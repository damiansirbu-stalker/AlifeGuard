# AlifeGuard Architecture

## Design

Single-pass population control. One timer (configurable interval, default 30s) collects all online entities via game_objects_iter, sorts by distance, releases farthest-first until under threshold.

## vs GAMMA Dynamic Despawner (Grok)

| | AlifeGuard | Grok |
|---|---|---|
| Collection | game_objects_iter (1 pass/check) | npc_on_update + monster_on_update (per entity per frame) |
| Removal order | Farthest-first (distance sorted) | Random (shuffle_table) |
| Protection | 4-layer engine API + cached lookups | Hardcoded 170+ section names |
| Release API | alife_release_id (direct) | alife_release(alife_object(id)) (extra fetch) |
| Task check | xcreature.is_task_giver (cached) + xsquad.is_task_target | generate_ongoing_tasks per NPC (expensive) |
| Timing | os.clock + configurable interval | time_global + global trigger variables |
| Globals | Zero (all local) | 6+ unscoped globals |

## Protection Layers

4 independent checks. Any single match keeps the entity alive.

| Layer | Source | What it covers |
|---|---|---|
| 1. Identity | xlibs xcreature.is_unscriptable | story_id, squad story_id, companions, named NPCs (traders, mechanics, leaders, medics, barmen, guides) |
| 2. Task giver | xlibs xcreature.is_task_giver | NPCs referenced by active tasks (by NPC ID or squad ID) |
| 3. Bounty/hostage | AG _is_bounty_or_hostage | axr_task_manager.bounties_by_id, hostages_by_id |
| 4. Task target | xlibs xsquad.is_task_target | Squads targeted by player tasks (assault, delivery, etc.) |

Layer 1 is cached per game_object with weak keys (GC-safe, zero repeated luabind).

## Performance

Designed to minimize luabind crossings (Lua -> C++ engine calls are expensive).

Collection:
- game_objects_iter: native C++ iterator from modded exes. No intermediate Lua table allocation.
- All engine APIs cached as local upvalues at module level (IsStalker, IsMonster, alife_object, alife_release_id). Eliminates global table lookups on every call.

Distance:
- distance_to_sqr throughout. Comparison against squared thresholds. No sqrt calls anywhere in the hot path.
- Sort uses squared distances directly (ordering is preserved: a^2 > b^2 iff a > b for positive distances).

Protection:
- xcreature.is_unscriptable: weak-key object cache. First call per entity costs 3-5 luabind (id, story_id, has_info, section, get_object_squad). Every subsequent call for the same object: 0 luabind (pure Lua table lookup). Cache entries auto-evict on GC.
- xcreature.is_task_giver: O(t) where t = active tasks (typically 3-10). 1-2 luabind per call.
- xsquad.is_task_target: O(t) same. 0-1 luabind per call.
- obj:section() gated behind _dbg flag. This luabind call is skipped entirely in production (non-debug) mode.

Release:
- alife_release_id(id) - single engine call. Grok uses alife_release(alife_object(id)) - two calls.
- xtable.sort: in-place quicksort, no allocation.

Total cost per check cycle: 1 game_objects_iter pass + 1 sort + N releases. N is typically 0-5. The entire cycle runs in <2ms on a typical GAMMA save (measured via xprofiler).

## Flow

```
actor_on_update (every frame)
  -> clock check (interval gate, skip if < check_interval)
  -> clock check (grace period gate, skip if < 30s after level load)
  -> count_and_collect_online()
     -> game_objects_iter: collect all stalkers + mutants
     -> per entity: distance_to_sqr, _is_entity_protected (4 layers)
  -> if over threshold:
     -> sort by distance (farthest first)
     -> release unprotected entities until under target
     -> notify via PDA
```

## Files

| File | Purpose |
|---|---|
| ag_population.script | Population control, protection, release, MCM buttons |
| ag_mcm.script | MCM configuration, defaults, UI definition |
| _ag_deps.script | Version, xlibs dependency gate |
