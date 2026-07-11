# AlifeGuard Architecture

Population control for STALKER Anomaly. Keeps online entity count under a configurable threshold by releasing NPCs back to offline simulation. Squad-aware: thins squad members before touching commanders, spreads removals evenly across factions and mutant types via round-robin, uses hysteresis to prevent oscillation. Releases are frame-spread (1 per frame via xslice) to bound `safe_release_manager`'s per-frame work and keep cleanup smooth.

Built on xlibs (xsquad, xcreature, xslice, xprofiler, xlog).

Part of a three-mod alife family: **AlifePlus** extends A-Life with new behaviors, **AlifeBalance** tunes existing rates and counts, **AlifeGuard** keeps alife state clean (this mod).

---

## Invariants

- **No steady-state per-frame work.** Ongoing work runs on a throttled tick (a fixed interval) or on a discrete engine event (hit, shot, spawn, option change); it never runs continuously every frame. A per-frame engine callback (`npc_on_update`) is used only as a carrier that throttles before doing anything, and we never place our code on a path the engine runs every frame (a visibility or fire functor). Frame-spreading a bounded one-off batch (xslice, 1 item per frame) to avoid a single-frame spike is the one allowed use of the frame; it completes and stops. Full rule and rationale: `doc/standards/code-standards.md` "No Per-Frame Work".

---

## Pipeline

One cycle per `check_interval` (default 30s). Two phases: synchronous collection (frame 0), frame-spread release (frames 1-N).

```
actor_on_update (every frame)
  |-- guard: cfg.enabled, db.actor, xslice not active, grace period, interval
  |
  v
FRAME 0: _collect_online
  |-- xcreature.online_iter_with_id() -> _process_entity per obj
  |     classify (stalker/mutant/skip)
  |     distance_to_sqr from actor
  |     resolve squad via alife_object(id) -> se_obj.group_id -> alife_object(group_id)
  |     group into squad records: commander, removable members, category
  |     orphans (no squad) -> separate list
  |
  |-- if total > max:
  |     target = max - buffer (hysteresis)
  |     _build_release_queue(squads, orphans, total - target)
  |       _categorize_squads -> 4 tiers
  |       _sort_categories -> farthest first per category
  |       _round_robin_fill -> interleave categories
  |
  v
FRAMES 1..N: xslice "ag_despawn", step=1
  |-- per frame: safe_release(entity_id) -> alife_release
  |-- on_done: log summary, PDA notification
```

---

## Squad-Aware Culling

### Why

Releasing individual NPCs by distance destroys entire squads. `alife_release_id(npc)` routes through `_g.script:alife_release` -> `squad:remove_npc(id, true)`. When `npc_count()` hits 0, the squad is deleted, `already_spawned` is decremented, and the originating smart terrain gets a free respawn slot. This creates a feedback loop: AG culls -> smart terrain respawns -> AG culls again.

### How

Collection groups entities by squad. Each squad record tracks:

| Field | Source | Purpose |
|---|---|---|
| commander_id | `squad:commander_id()` | Lowest member ID in engine's `m_members` (associative_vector) |
| protected | `xsquad.is_protected()` | Squad-level protection (story, trader, task giver, companion, bounty/hostage) |
| scripted | `xsquad.is_scripted()` | Has scripted_target, action_condlist, or random_targets |
| category | `squad.player_id` | Faction or mutant type (0 luabind, Lua field) |
| removable | collected | Non-commander, non-protected members eligible for release |

Keeping the commander alive means the squad survives. `already_spawned` stays unchanged. All spawn paths are blocked:

| Spawn path | Gate | Blocked by lone commander? |
|---|---|---|
| `try_respawn` | `already_spawned[section].num < max_respawn_count` | Yes - counter not decremented |
| Warfare | `squad_count(smart, faction) < totalSize` | Yes - squad still in SIMBOARD.squads |
| ZCP | Via try_respawn gates | Yes |
| SIMBOARD routing | `population >= max_population` | Yes - squad still assigned |

---

## Protection

Two layers. Squad-level first, per-member fallback for named NPCs.

### Squad-level: xsquad.is_protected

Single call per squad. Checks (in order):

1. **Permanent** (cached, session lifetime): story_id, trader, named NPC commander, empty squad
2. **Active role** (dynamic, if `check_tasks` enabled): task_giver, companion
3. **Task target** (dynamic, if `check_tasks` enabled): task_squads hash + member bounty/hostage

Protected results cached with TTL (120s, positive-only). Uncached squads always get live check. False negatives impossible. False positives harmless (squad stays online slightly longer, clears on TTL expiry). Cache cleared on MCM config change.

`is_scripted` is NOT a protection check. Scripted squads are cullable but deprioritized (tier 2/4 instead of 1/3).

### Per-member: xcreature.is_unscriptable

After squad-level check passes, each non-commander member is checked individually. Catches named NPCs (traders, mechanics, guides) who are not the squad commander. Weak-key cache, session lifetime, 0 luabind on hit.

---

## Priority Tiers

Entities are sorted into 4 tiers. Each tier is fully exhausted before the next is touched.

| Tier | Contents | Rationale |
|---|---|---|
| 1 | Non-commander members from unscripted squads + orphans | No mod dependency, expendable |
| 2 | Non-commander members from scripted squads | Some mod set scripted_target, but members are expendable |
| 3 | Commanders from unscripted squads | Kills the squad, opens respawn slot |
| 4 | Commanders from scripted squads | Last resort: kills a mod-controlled squad |

Most cycles never leave tier 1. Heavy load reaches tier 2. Tiers 3-4 are edge cases (more lone-commander squads online than max).

---

## Round-Robin Fairness

Within each tier, entities are grouped by category (`squad.player_id`: "bandit", "duty", "monster_predatory_day", etc.). Orphans form their own category.

Each category's entries are sorted farthest-first (or shuffled if `prioritize_distant` is off). The queue is built by cycling through categories, taking 1 entity per category per round.

```
Round 1: bandit(far) -1 | duty(far) -1 | boar(far) -1 | snork(far) -1
Round 2: bandit(far) -1 | duty(far) -1 | boar(far) -1 | snork(far) -1
...until target reached or all categories exhausted
```

Categories with fewer members exhaust first and are skipped in subsequent rounds. Large populations lose more in absolute numbers but less per-capita.

---

## Hysteresis

Prevents oscillation between cull cycles.

- **Trigger**: `total > max` (e.g., 81 > 80)
- **Target**: `max - buffer` (e.g., 80 - 10 = 70)
- **Next trigger**: not until `total > max` again

Without hysteresis: cull to 80, 2 NPCs respawn, cull again next cycle. With buffer=10: cull to 70, population rises to 78, no cull. Only triggers when it crosses 81 again.

---

## Frame-Spread Release

xslice (xlibs) processes the release queue at 1 entity per frame via `AddUniqueCall`. Not multithreading -- cooperative time-slicing on X-Ray's single Lua thread.

Why 1 per frame: `safe_release_manager` (Alundaio, `safe_release_manager.script:4-7`) handles the binder-still-alive race between `release` and `net_destroy` by deferring each entity's actual release across several frames (`set_switch_online(false)` + `set_switch_offline(true)` + `switch_offline()`, then `sim:release` once the binder is gone). It drains its `objects_to_release` dict via `AddUniqueCall` once per frame and walks all pending entries on each pass. A synchronous N-entity release loop pushes N entries in one frame, so every subsequent drain pass does N `switch_offline` calls until the binders finish. xslice's 1-per-frame pacing keeps `objects_to_release` at depth 1, so per-frame cost stays flat. Intra-slice deaths (entity died between collection and release) fail the `alife_object(id)` verify and are skipped at zero cost.

Release path per entity:
```
safe_release(id)
  -> alife_object(id)       -- verify exists (1 luabind medium)
  -> alife_release(se)      -- _g.script routes to squad:remove_npc(id, true)
    -> smart:unregister_npc(squad)
    -> squad:unregister_member(id)
    -> safe_release_manager.release(se_obj)
    -> if npc_count > 0: squad:refresh() (commander auto-promotes, map marker updates)
    -> if npc_count == 0: full squad cleanup, on_unregister, already_spawned decrement
```

Stale entity IDs (entity died between collection and release): `alife_object(id)` returns nil, `safe_release` returns false, entity skipped.

---

## Smart Sanitizer

### Why

`smart.already_spawned[k].num` underflows when more than one release path decrements the counter. Source: engine `sim_squad_scripted.script:1028` (squad on_unregister) racing with external despawners (e.g. Grok's `grok_dynamic_despawner.script:289` calling `alife_release`). Two consequences:

1. Save CTD. STATE_Write at `smart_terrain.script:955` casts `num` to `u8`. Negative values produce `CRITICAL ERROR: write u8 (-1)`.
2. Infinite spawn. Respawn gate at `smart_terrain.script:1696` reads `max > num`. Negative `num` makes the comparison always true; the smart spawns every cycle without bound.

### How

Periodic walk over `SIMBOARD.smarts` clamping three invariant violations on each entry of `smart.already_spawned`:

| Case | Detection | Fix |
|---|---|---|
| A | `type(v) ~= "table"` | `smart.already_spawned[k] = { num = 0 }` |
| B | `type(v.num) ~= "number"` | `v.num = 0` |
| C | `v.num < 0` | `v.num = 0` |

Hooks:
- `actor_on_reinit` fires after STATE_Read (data live) and before `bind_stalker.script:200` runs `set_objects_per_update(65534)` (engine load burst). Cleans corrupted save data before `try_respawn` reads it.
- `actor_on_update` interval gate fires every 300 seconds during play. Catches mid-session corruption from misbehaving mods still installed.

### Cost

Pure Lua walk. 0 luabind in the inner loop. `smart:name()` is lazy: called once per smart only if a fix fires on that smart. Sub-millisecond for 50-200 smarts in a healthy save (no fixes). Worst case (every smart corrupted, 200 smarts): ~2ms.

---

## Offline Guard (ag_density)

### Why

`_collect_online` sees only online entities. An offline hub on another level can accumulate dozens of squads that all come online at once on fast-travel, spiking well past `max` before the online guard's first cycle. The Offline Guard measures offline density per region and shaves badly crowded regions to the online ceiling before the player arrives.

### Scan

Time-event tick (`CreateTimeEvent`/`ResetTimeEvent`, no per-frame carrier), `density_interval` seconds apart. Each tick bins 25 squads from a `xsquad.collect_squad_ids` snapshot via a circular cursor; a full pass spreads across ticks at constant cost regardless of total squad count.

Per squad: skip online, skip empty, skip the actor's level; bin `squad:npc_count()` into a cell keyed `level_id:floor(x/D):floor(z/D)` with `D = sim:switch_distance()`. The squad's own `position`/`m_game_vertex_id` is used directly: offline, the engine moves the group and syncs every member's position FROM it (`CSE_ALifeOnlineOfflineGroup::update`, alife_online_offline_group.cpp:73-87), so the group fields are the bodies' physical position. `[DENSITY]` lines at DEBUG per completed pass.

### Cull

At pass end, cells with `count > density_trigger` on non-actor levels are thinned by `count - density_target` members. The synchronous step is only a pure-Lua work list: each over-trigger cell contributes a shared budget table (`count - target`) and one `{squad_id, budget}` item per squad, with zero luabind. All squad resolution, protection, and release is deferred into the shared xslice `"ag_despawn"` job, one squad per frame, so no single frame examines more than one squad and the build stays flat regardless of cull size.

Per squad in its frame: resolve fresh, skip if gone/online/lone/protected (`xsquad.is_protected`), then release its non-commander members down to the cell budget. Commanders are never queued. Per-member protection uses `xcreature.is_unscriptable_se`, the offline-safe variant of the online path's `is_unscriptable` (story-id registry, `xdata.unscriptable_npcs` section hash, squad story-id — everything the online check reads except the game-object-only companion flag). Task-giver and bounty/hostage protection is squad-level via `is_protected`, gated by the offline guard's own `density_check_tasks`.

No round-robin across factions (that needs the whole member set built at once, which defeats frame-spreading). It is unnecessary offline: every squad keeps its commander and survives, so no faction is wiped and there is nothing to spread fairly. Cells are processed in scan order. The `xslice.is_active` guard means the offline and online culls never run despawn jobs concurrently; a cull that outlasts the next scan pass simply defers the next cull until it finishes.

Fully decoupled from the online guard. `density_trigger` (when a region is overcrowded) and `density_target` (what to thin it to) are absolute body counts, not derived from the online `max`. Target can go as low as 0 (strip a region to lone commanders), so offline can be culled harder than the online cap if wanted. A target above the trigger is clamped to the trigger. The offline pass reads none of the online guard's keys.

### Release path (offline)

The squad is resolved fresh in its own drain frame, so member id and commander id are current — there is no build/drain staleness window (the squad could have gone online, dissolved, or lost its commander between the pass end and this frame, all caught by the fresh `_cull_squad` resolve). Per member: skip if it is the commander, gone, online, or named/story (`is_unscriptable_se`). Then, in order:

1. `smart:unregister_npc(se)` with the NPC when `smart_terrain_id() ~= 65535` - clears the smart's `npc_info[npc_id]`/`arriving_npc[npc_id]` by the correct key and sets `m_smart_terrain_id = 0xffff`. Vanilla `remove_npc` passes the squad instead, which would leave a stale `npc_info` entry holding a destroyed server-object reference.
2. `alife_release(se)` - routes to `squad:remove_npc(id, true)`: `unregister_member` detaches the engine-side member pointer BEFORE `safe_release_manager` destroys the entity. Offline entities have no binder, so the release completes on the manager's next pass.

A squad's surplus members release together in its one frame (bounded by squad size, ~9 max, all offline so no `switch_offline` dance and no binder wait). One squad per frame keeps per-frame work bounded and the synchronous build off the hot path.

Direct `alife():release()` on a squad member is forbidden: the group's `update()` writes through raw `m_members` pointers and the engine has no member auto-detach on release (use-after-free).

Keeping the commander keeps `npc_count() > 0`, so `remove_npc` never reaches its squad-deletion branch: the squad stays in SIMBOARD, `already_spawned` stays untouched, no respawn feedback.

Full design record with engine citations: `stalker-dev/doc/todo/todo-alifeguard-offline-density.md`.

---

## Performance

### Collection (frame 0, synchronous)

| Operation | Cost per entity | Notes |
|---|---|---|
| obj:id(), IsStalker/IsMonster, alive(), position() | 5 luabind trivial | Unavoidable |
| distance_to_sqr | 1 luabind trivial | Squared distances, no sqrt |
| alife_object(id) | 1 luabind medium | For group_id lookup |
| se_obj.group_id | 1 luabind trivial | def_readonly on CSE_ALifeMonsterAbstract |
| alife_object(group_id) | 1 luabind medium | Squad resolution (skip if 65535) |
| xcreature.is_unscriptable | 0 luabind (cached) | Weak-key, session lifetime |
| xsquad.is_protected | 0 luabind (TTL cached) | Per squad, not per member |
| obj:section() | 1 luabind heavy | Per collected entry; stored for the release-time id-recycle check |

Total: ~9-10 luabind per entity (production), ~1-6 per squad. Sub-millisecond for 100-200 entities.

### Release (frames 1-N)

2 luabind per frame (alife_object verify + alife_release call). ~0.05ms per release. 30 excess entities = 30 frames = 0.5s at 60fps.

### Benchmarks

Playtested: Army Warehouses, 83 online, threshold 50, 33 removed across 40 frames. Per-release 0.05ms avg. Zero stutter. See `doc/img/benchmark_despawn_spread.jpg`.

---

## Files

| File | Lines | Purpose |
|---|---|---|
| ag_population.script | 539 | Collection, squad grouping, tier/round-robin queue, frame-spread release, smart sanitizer |
| ag_density.script | 314 | Offline density scan per switch_distance cell, per-cell offline cull |
| ag_mcm.script | 210 | MCM defaults, UI definition, button handlers |
| _ag_deps.script | 121 | Version string, xlibs + modded-exes/AOEngine dependency gate, platform status footer |
| ag_test.script | 180 | Dormant console load/conformity harness for the offline guard |

---

## MCM Settings

| Setting | Default | Effect |
|---|---|---|
| enabled | true | Master toggle |
| max | 80 | Trigger threshold (cull when total exceeds this) |
| buffer | 10 | Hysteresis gap (cull target = max - buffer) |
| check_interval | 30 | Seconds between checks |
| check_tasks | true | Protect task givers, companions, bounty/hostage targets |
| prioritize_distant | true | Farthest first (false = random) |
| squad_culling | true | Thin squad members before commanders (false = flat by distance) |
| round_robin | true | Spread removals evenly across factions and mutant types (false = linear) |
| pda | true | PDA notifications on cleanup |
| pda_sound | true | Play a sound with the cleanup notification |
| density_enabled | true | Offline Guard master toggle |
| density_trigger | 120 | Offline body count that flags a region as overcrowded (absolute, own setting) |
| density_target | 80 | Body count a flagged region is thinned to (0-300, clamped to trigger; 0 = lone commanders) |
| density_check_tasks | true | Protect task givers, bounty/hostage targets in the offline pass (own toggle) |
| density_interval | 1 | Seconds between offline scan steps (1-10) |
| sanitize_smarts | true | Periodic walk that clamps corrupted already_spawned counters |
| sanitize_interval | 300 | Seconds between periodic sanitizer passes (60-1800) |
| log_level | WARN | Logger verbosity (ERROR/WARN/INFO/DEBUG) |
