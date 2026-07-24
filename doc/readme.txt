AlifeGuard: A-Life population governor for STALKER Anomaly, by Damian
Version: 1.3.1 (xlibs 1.8.3, demonized 20250908)
GitHub: https://github.com/damiansirbu-stalker/AlifeGuard
Changelog: https://github.com/damiansirbu-stalker/AlifeGuard/blob/main/doc/changelog
Russian / Na russkom: https://github.com/damiansirbu-stalker/AlifeGuard/blob/main/doc/readme_ru.txt
Bugs, suggestions: https://github.com/damiansirbu-stalker/AlifeGuard/issues

Alife Collection:
AlifePlus: https://www.moddb.com/mods/stalker-anomaly/addons/alifeplus-v1-0-01
AlifeBalance: https://www.moddb.com/mods/stalker-anomaly/addons/alifebalance
AlifeGuard: https://www.moddb.com/mods/stalker-anomaly/addons/alifeguard-1001
AlifeTactics: https://www.moddb.com/mods/stalker-anomaly/addons/alifetactics

! Reset MCM settings to defaults after updating !

Late-game A-Life accumulates too many active entities. AI, physics, and pathfinding all
run on the same thread. Performance degrades as the count grows. Population mods like
ZCP and Redone amplify the problem by increasing spawn rates. Zombie entities from broken
releases, orphaned squad members, and engine-level memory leaks make it worse over time.

AlifeGuard keeps the online population under control while preserving how A-Life works.
Squads remain valid, smart terrains behave correctly, no respawn loops. It reduces load
without breaking the simulation.

Simple despawners remove individual NPCs without considering squad structure. This can
delete entire squads, open respawn slots, leave smart terrains in a malformed state where
they stop spawning, and interfere with mods that script squads. AlifeGuard works at the
squad level and accounts for the engine's spawn bookkeeping.

Squad-aware culling:
  Entities are processed as part of their squad, not individually. Non-commander members
  are thinned first. Commanders are removed only as a last resort.
  This preserves squads in SIMBOARD, keeps already_spawned counters intact, and prevents
  the repeated respawns that individual-NPC despawners cause.
  Squads controlled by other mods (AlifePlus, Warfare, Guards Spawner) are deprioritized:
  their members go after unscripted members, their commanders last of all.

Round-robin fairness:
  Removals are spread across factions and mutant types. One entity per category per round.
  No single group is disproportionately affected.

Hysteresis:
  Separate trigger and target thresholds (default: trigger at 80, cull to 70). Cleanup runs
  in cycles, not constantly reacting to small population changes.

Frame-spread release:
  Entities are released one per frame via xslice (cooperative time-slicing). 30 excess
  entities = 30 frames to clear. No spikes, no freezes.

Protection:
  Story squads, traders, named NPCs, companions, task givers, bounty and hostage targets
  are never removed. Squad-level checks with positive-only TTL cache. Per-member fallback
  catches named NPCs who are not squad commanders.

Offline Guard:
  The online guard only sees entities near you. A hub on another level can hold dozens of
  offline squads that all come online at once and spike the count. The Offline Guard runs a
  staggered scan that bins offline squads into regions by position and thins any region over
  the trigger down to the target. Commanders and protected squads are never removed, so every
  squad survives. Independent of the online guard, with its own trigger, target, and task
  protection. Target down to 0 strips a region to lone commanders. Toggle, cull trigger, cull
  target, task protection, and scan tick in MCM.

Smart Sanitizer:
  Defensive pass that clamps corrupted respawn counters on smart terrains. Negative or
  non-numeric already_spawned values cause save crashes (u8 write overflow at STATE_Write)
  and infinite spawn loops (max > num respawn gate always true). The sanitizer runs on
  actor_on_reinit before the load-time alife burst, then every 300 seconds during play.
  Lua table walk over SIMBOARD.smarts. Sub-millisecond on 50-200 smarts. Toggle and
  interval in MCM.

Inventory Guard:
  Vanilla NPCs loot corpses, and over a long session this builds up two costs.
  A long-lived stalker carries a trader run of gear, so killing them becomes a jackpot.
  Every looted item is also a permanent alife server object, and the engine tracks at most around 65000 of those.
  Long saves drift toward the cap and the game eventually slows, then crashes.

  Why not just disable NPC looting?
    Mods like NPC Stop Looting Dead Bodies and BoltBeGone sidestep the problem by blocking the engine's loot path.
    That fixes the symptom but stalkers no longer loot their kills, which is one of the things that makes A-Life feel alive.
    Inventory Guard keeps vanilla looting on and bounds the cause instead.
    A lightweight scanner walks online stalkers in small batches and releases anything above each category's limit.
    Killing a stalker still yields what they actually need to carry, not what they have accumulated over 50 game-days.

  What you'll notice:
    Long-lived stalkers carry a believable loadout instead of a trader-sized hoard.
    Killing a random stalker yields reasonable loot, not a vendor run.
    Saves stay performant across long sessions.
    Companions, story characters, and named NPCs are never touched.
    Traders, mechanics, and medics are matched by role, so even ones spawned at runtime (Warfare and similar) skip the per-category trim; only the trader stock purge below applies to them.
    Vanilla looting still works. Stalkers loot their kills.

  Important:
    Inventory Guard never spawns items.
    It releases what NPCs already accumulated above the per-category limits.
    Quest items, equipped gear, story_id items, companion gifts, and player-strapped weapons are always preserved.

  Example:
    An online stalker on Cordon has been alive for 50 game-days.
    They have picked up 47 medkits, 23 bandages, 18 grenades, and 600 rounds of mismatched ammo.
    The scanner's next visit releases 42 medkits (cap 5), 18 bandages (cap 5), 15 grenades (cap 3), and the 600 mismatched rounds (cap 0).
    The NPC walks around with a believable load: a few medkits, a stack of bandages, three grenades, and ammo for the gun they actually carry.

  Policy:
    Per-category limits live in configs/alifeguard/ag_inventory_policy.ltx (DLTX-overridable).
    Defaults cover equipped ammo (in rounds, per tier), grenades, and consumables: medkit, bandage, antirad, stim, pill, food, drink.
    Gear coverage: weapons, outfits, helmets, artefacts, crafting items, devices.
    Quest items, equipped gear, story_id items, companion gifts, and player-strapped weapons are protected by xlibs and never touched.

  Trader stock purge:
    Selling to a trader piles your junk into their inventory, and each item there is a permanent alife server object.
    The engine clears trader stock at restock, but between restocks a heavy selling session walks the object count toward the cap.
    The same scanner visits traders, mechanics, medics, and barmen and releases weapons, outfits, and helmets below a condition threshold (default 0.75).
    Stock at or above the threshold stays for sale; the trader's own equipped gear and quest items are never touched.
    Consumables, ammo, and money are never touched: only wear-bearing gear is judged by condition.
    Toggle and threshold in MCM.

  Moved from AlifeBalance (where it was called Inventory Balance). Same behavior, new home.
  If updating both mods, configure it here; the AlifeBalance tab is gone.

Performance:
  Single-pass collection (native C++ iterator). Cached protection lookups. Sub-millisecond
  scan for 200 entities. 0.05ms per release. Zero debug overhead when log level < DEBUG.

Mod compatibility:
  Most population mods conflict with A-Life mods. AlifeGuard is designed to coexist with
  AlifePlus, Warfare, ZCP, GAMMA, Guards Spawner, and any mod that uses scripted_target.
  Scripted squads are preserved as long as possible. Commanders remain, so squad assignments
  continue. Spawn counters stay consistent. Warfare population tracking stays accurate.
  AlifePlus cause/consequence chains complete because target squads persist.
  Other mods direct the world. AlifeGuard keeps it performant.

MCM:
  Online Guard: population limits, hysteresis buffer, check interval, protection rules
  (task NPCs, farthest-first, per-squad culling, round-robin), PDA notifications.
  Offline Guard: toggle, cull trigger, cull target, task protection, scan tick.
  Smart Sanitizer: toggle, interval.
  Inventory Guard: toggle, NPCs per frame, rescan cooldown, trader purge toggle, trader condition threshold.
  Development: log level (ERROR/WARN/INFO/DEBUG), diagnostics, population reset.

Requirements:
Anomaly 1.5.3
xlibs (https://www.moddb.com/mods/stalker-anomaly/addons/xlibs-1001)
MCM

Install (MO2):
1. Install xlibs
2. Install AlifeGuard
3. Load order does not matter
4. Configure via MCM

Uninstall (MO2):
Disable or remove in MO2.

Compatibility:
Requires xlibs.
Runs on themrdemonized modded exes 2025.9.10 or newer, or AOEngine v0.55 or newer.
The full feature set needs the latest demonized build. A feature that needs a newer build stays inactive on older exes.
Does not modify base scripts. Uses the standard engine API (alife_release).
Superseded (AlifeGuard does this - drop the other):
- Grok's Dynamic Despawner: same job, but leaks memory (table.remove during iteration) and drops squad commanders, forcing repeated respawns.
- Anti-loot addons (NPC Stop Looting Dead Bodies, BoltBeGone): Inventory Guard handles NPC looting at the source.

Conflicts (pick one - two governors fight):
- Any other despawn / population-release mod: conflicting releases, repeated respawns, entity leaks.
- Extended sim-distance / offline-range mods (Living Zone 2000m, Extended Offline, ROAD alife range 666): expand the population while AlifeGuard contracts it; they oscillate.

Affects / coexists:
- A-Life config tweaks (alife.ltx, smart max_population, Redone Alife Performance, x3 perf): engine-parameter layer, no overlap.
- AlifeBalance: different axis (respawn acceleration vs release work); composes.
- Weapons Drop on Bodies: only moves the dying NPC's weapon (corpse vs floor); doesn't block looting.

Known issue:
Rare crash on entity release (Perform_reject assertion). Engine-level issue in X-Ray's
inventory parent tracking, present in all population mods. No script-side fix exists.

Architecture:
- AlifeGuard runs on xlibs, a reverse-engineered API that wraps the X-Ray engine source.
  Squad lifecycle, protection checks, and spawn bookkeeping were traced through the C++ source.
  Core patterns were studied from the most accomplished mods in the Anomaly ecosystem.
- No base script edits, no engine patches. Runtime callbacks only.
- Two-phase pipeline: synchronous collection (frame 0), frame-spread release (frames 1-N).
  See doc/architecture.md for full design documentation.

Performance:
- Performance was a design constraint from the start.
  Collection: sub-millisecond for 200 entities. Release: 0.05ms per frame.
- Cooperative time-slicing (xslice) bounds per-frame release work and prevents frame
  stutter under heavy cleanup.
- Structured tracing with trace IDs, per-phase timing, null object singletons for zero
  debug overhead when log level < DEBUG.

Multi-stage validation pipeline:
- luacheck, selene (static analysis)
- tree-sitter AST analysis, ast-grep structural patterns
- Contract rules (API safety, cross-file dependencies, cyclomatic complexity, coding standards)
- lua54 integration testing with X-Ray engine stubs
- gitleaks (secret scanning)
Full report in doc/test-report.log.

FAQ:
Do I need modded exes?
  Yes. AlifeGuard needs themrdemonized modded exes (2025.9.10 or newer) or AOEngine (v0.55 or newer). Vanilla Anomaly does not expose the APIs it relies on.

Credits:
Altogolik - support, ideas, source materials

Usage and License:
  Modpacks: allowed and encouraged. Keep the readme and license files.
  Addons, patches, integrations: allowed. Credit "AlifeGuard by Damian Sirbu" visibly on your mod page.
  Reproducing the implementation in other software: not allowed, even with credit.
  Full license in LICENSE file and on GitHub.

Reporting issues and suggestions
Open a bug report or a suggestion at https://github.com/damiansirbu-stalker/AlifeGuard/issues/new/choose.
Also discussed on the GAMMA, EFP, Anomaly, and Zona Discord servers.

Before posting, read this readme and the MCM options.

Include:
- Exact steps to reproduce, from a new game or a named save, with expected and actual result.
- xray.log and the mod debug log (MCM log level DEBUG), plus engine build, modlist, load order.
- Describe the behavior. With hundreds of mods and overrides, only the log shows whether this mod was involved and what caused it.
