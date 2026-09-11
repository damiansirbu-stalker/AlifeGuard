AlifeGuard: A-Life performance and stability for STALKER Anomaly, by Damian
Version: next (xlibs 1.8.3, demonized 20250908)
GitHub: https://github.com/damiansirbu-stalker/AlifeGuard
Changelog: https://github.com/damiansirbu-stalker/AlifeGuard/blob/main/doc/changelog
Russian / Na russkom: https://github.com/damiansirbu-stalker/AlifeGuard/blob/main/doc/readme_ru.txt
Bugs, suggestions: https://github.com/damiansirbu-stalker/AlifeGuard/issues

Alife Collection:
AlifeAmbience: https://github.com/damiansirbu-stalker/AlifeAmbience
AlifeBalance: https://www.moddb.com/mods/stalker-anomaly/addons/alifebalance
AlifeCompanions: https://github.com/damiansirbu-stalker/AlifeCompanions
AlifeDiegetic: https://www.moddb.com/mods/stalker-anomaly/addons/diegetic-audio-control-100
AlifeGuard: https://www.moddb.com/mods/stalker-anomaly/addons/alifeguard-1001
AlifePlus: https://www.moddb.com/mods/stalker-anomaly/addons/alifeplus-v1-0-01
AlifeSpooks: https://github.com/damiansirbu-stalker/AlifeSpooks
AlifeTactics: https://www.moddb.com/mods/stalker-anomaly/addons/alifetactics
FurnitureFuel: https://github.com/damiansirbu-stalker/FurnitureFuel
JitProfiler: https://github.com/damiansirbu-stalker/JitProfiler
TestZone: https://github.com/damiansirbu-stalker/TestZone
xlibs: https://www.moddb.com/mods/stalker-anomaly/addons/xlibs-1001

! Reset MCM settings to defaults after updating !

Late-game A-Life accumulates too many active entities. AI, physics, and pathfinding all run on the same thread, so performance degrades as the count grows.
Population mods like ZCP and Redone amplify the problem by raising spawn rates. Zombie entities from broken releases, orphaned squad members, and engine memory leaks make it worse over time.

AlifeGuard keeps house over the world's NPCs, creatures, and items, catching what would corrupt a save, break data, or drag performance before it does.
Squads stay valid and smart terrains behave correctly, with no respawn loops. It reduces load without breaking the simulation.

Simple despawners remove individual NPCs without considering squad structure. This deletes entire squads and opens respawn slots.
It leaves smart terrains unable to spawn and interferes with mods that script squads. AlifeGuard works at the squad level and accounts for the engine's spawn bookkeeping.

Systems:
- Online Guard - caps the online population near you: when the count crosses the trigger it culls back to the target, squad by squad.
- Offline Guard - a staggered scan bins offline squads by region and thins the crowded ones, so a hub full of offline squads cannot spike the count when it wakes. Commanders always survive.
- Smart Sanitizer - clamps corrupted respawn counters on smart terrains, the kind that cause save crashes and infinite spawn loops.
- Inventory Guard - bounds what NPCs hoard from looting. Vanilla looting stays on, but a long-lived stalker carries a believable load instead of a trader run, and looted items stop drifting toward the engine's ~65000-object cap that crashes long saves.

Mechanisms (how the culling stays safe):
- Squad-aware - thins non-commanders first, commanders last, so squads stay valid in SIMBOARD and no respawn loop starts. Mod-owned squads (AlifePlus, Warfare, Guards Spawner) go last.
- Round-robin - removals spread across factions and mutant types, one per category per round, so no group is thinned disproportionately.
- Hysteresis - separate trigger and target thresholds (default 80 down to 70), so cleanup runs in cycles, not on every small change.
- Frame-spread - entities released one per frame via xslice, so a cleanup spreads over frames and never hitches the game.
- Protection - story squads, traders, named NPCs, companions, task givers, quest/bounty/hostage targets, and scripted squads are never touched. Toggle scripted protection off for the most aggressive culling.

Requirements:
Anomaly 1.5.3
Modded exes: themrdemonized 2025.9.10 or newer, or AOEngine v0.55 or newer. The full feature set needs the latest demonized build; a feature that needs a newer one stays inactive on older exes.
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
Coexists with AlifePlus, Warfare, ZCP, Guards Spawner, AlifeBalance, and any mod using scripted_target. Scripted squads are protected by default.
- Supersedes: Grok's Dynamic Despawner.
- Redundant: anti-loot addons (NPC Stop Looting Dead Bodies, BoltBeGone) - AlifeGuard bounds looting while keeping it on.
- Conflicts: any other despawn or population-release mod; extended sim-distance mods (Living Zone 2000m, Extended Offline, ROAD range) that push population up while AlifeGuard pulls it down.

Known issue:
A rare crash on entity release (Perform_reject assertion). This is an engine-level fault in X-Ray inventory parent tracking, present in all population mods, with no script-side fix.

Performance:
Performance comes first, ahead of any feature.
When a feature cannot fit the budget it is reworked or removed, even with an X-Ray engine modification. It is never allowed to slow the game.
Collection is a single pass over a native C++ iterator with cached protection lookups, sub-millisecond for 200 entities.
Release costs 0.05ms, and debug work drops to nothing when the log level is below DEBUG.
The timings are measured on the engine built from the latest source with no multithreading and no optimizations, so they are worst-case.
The optimized multithreaded build you run is always faster.

Development:
- AlifeGuard runs on xlibs, a reverse-engineered API that wraps the X-Ray engine source. Squad lifecycle, protection checks, and spawn bookkeeping were traced through the C++ source.
- Core design patterns were studied from the most accomplished mods in the Anomaly ecosystem.
- No base script edits, no engine patches, runtime callbacks only.
- A two-phase pipeline runs synchronous collection on frame 0, then frame-spread release over frames 1-N. Cooperative time-slicing (xslice) bounds per-frame release work and prevents frame stutter under heavy cleanup.
- Structured tracing carries trace IDs and per-phase timing, and null object singletons drop debug overhead to nothing when the log level is below DEBUG.
- Validated by a multi-stage pipeline: luacheck and selene (static analysis), tree-sitter AST analysis and ast-grep structural patterns, contract rules (API safety, cross-file dependencies, cyclomatic complexity, coding standards), lua54 integration testing with X-Ray engine stubs, and gitleaks secret scanning. The full report lives in doc/test-report.log. See doc/architecture.md for the full design.

Credits:
Altogolik - support, ideas, source materials

Usage and License:
Modpacks are allowed and encouraged. Keep the readme and license files.
Addons, patches, and integrations are allowed. Credit "AlifeGuard by Damian Sirbu" visibly on your mod page.
Reproducing the implementation in other software is not allowed, even with credit. The full license lives in the LICENSE file and on GitHub.

Diagnostics and reporting:
Development > Log level: set to DEBUG, reproduce, then back to WARN. Writes the debug log.
Report at https://github.com/damiansirbu-stalker/AlifeGuard/issues/new/choose or the EFP, Anomaly, and Zona Discord. Include repro steps, engine build, modlist, load order, xray.log, and the debug log.
