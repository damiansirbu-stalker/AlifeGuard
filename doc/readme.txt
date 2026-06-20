AlifeGuard: A-Life population governor for STALKER Anomaly, by Damian
Version: 1.2.7 (xlibs 1.7.7)
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
  the respawn churn that individual-NPC despawners cause.
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

Smart Sanitizer:
  Defensive pass that clamps corrupted respawn counters on smart terrains. Negative or
  non-numeric already_spawned values cause save crashes (u8 write overflow at STATE_Write)
  and infinite spawn loops (max > num respawn gate always true). The sanitizer runs on
  actor_on_reinit before the load-time alife burst, then every 300 seconds during play.
  Lua table walk over SIMBOARD.smarts. Sub-millisecond on 50-200 smarts. Toggle and
  interval in MCM.

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
  General: population limits, hysteresis buffer, check interval, protection rules
  (task NPCs, farthest-first, per-squad culling, round-robin), PDA notifications,
  Smart Sanitizer (toggle, interval).
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
- Grok's Dynamic Despawner: same job, but leaks memory (table.remove during iteration) and drops squad commanders, churning respawns.

Conflicts (pick one - two governors fight):
- Any other despawn / population-release mod: conflicting releases, respawn churn, entity leaks.
- Extended sim-distance / offline-range mods (Living Zone 2000m, Extended Offline, ROAD alife range 666): expand the population while AlifeGuard contracts it; they oscillate.

Affects / coexists:
- A-Life config tweaks (alife.ltx, smart max_population, Redone Alife Performance, x3 perf): engine-parameter layer, no overlap.
- AlifeBalance: different axis (respawn acceleration vs distant-NPC release); composes.

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
