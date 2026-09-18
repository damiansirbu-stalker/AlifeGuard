AlifeGuard: A-Life performance and stability for STALKER Anomaly, by Damian
Version: 1.3.2-snapshot (xlibs 1.8.3, demonized 20250908)
Changelog: https://github.com/damiansirbu-stalker/AlifeGuard/blob/main/doc/changelog
Russian / Na russkom: https://github.com/damiansirbu-stalker/AlifeGuard/blob/main/doc/readme_ru.txt

My work:
GitHub: https://github.com/orgs/damiansirbu-stalker/repositories
ModDB: https://www.moddb.com/members/damian-sirbu/addons
Nexus: https://www.nexusmods.com/profile/damiansirbu/mods

My contributions:
X-Ray Monolith: https://github.com/themrdemonized/xray-monolith

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
Modded exes: themrdemonized 20250908 or newer, or AOEngine v0.55 or newer. The full feature set needs the latest demonized build. A feature that needs a newer one stays inactive on older exes.
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

How It's Built:

Although it started from work by Demonized, Alundaio, and Tronex, the current code and patterns are original, learned through reverse-engineering X-Ray, load testing, and custom X-Ray changes.
The design favors the engine's own mechanisms and minimal intervention, with event-native pub/sub over polling.
Work spreads across frames through deferred queues and rate limiters, while per-level caches replace world scans.
The raycasting and range math are hand-written and tested live, and the code follows the engine's own standards and flags.
Performance is the first invariant. Every flow stays under 2ms, and the build rewrites or drops anything that misses.
Profiled continuously with JitProfiler, an engine-native scientific tool. Manual tests run on unoptimized, single-threaded exes.
The code carries tracing and monitoring from the ground up, with every flow timed off the log level.
Every commit runs the full pipeline locally and in CI: luacheck, a Selene build compiled for STALKER with flags the public build lacks, and a load test that runs every script against engine stubs.
Rule layers then check crash safety, hotpath cost, engine correctness, complexity, architecture contracts, security, and the docs.
Every mod is configurable through MCM or LTX, down to each rate, threshold, and toggle, with nothing tunable left hard-coded.
The mod avoids writing engine values, holding its own state in parallel. Any value it must change stays inside the engine's own bounds, so save corruption is impossible.
It depends on no other mod, not even my own. The only shared layers are X-Ray and xlibs.

[Screenshot: AlifeGuard under JitProfiler, a live CPU and allocation capture]
Project Health: https://damiansirbu-stalker.github.io/AlifeGuard/

Credits:
Altogolik - support, ideas, source materials

Usage and License:
  Modpacks: allowed and encouraged. Keep the readme and license files.
  Addons, patches, integrations: allowed. Credit "AlifeGuard by Damian Sirbu" visibly on your mod page.
  Reproducing the implementation in other software: not allowed, even with credit.
  Full license in LICENSE file and on GitHub.

Diagnostics and reporting:
Development > Log level: set to DEBUG, reproduce, then back to WARN. Writes the debug log.
Report at https://github.com/damiansirbu-stalker/AlifeGuard/issues/new/choose or the EFP, Anomaly, and Zona Discord. Include repro steps, engine build, modlist, load order, xray.log, and the debug log.
