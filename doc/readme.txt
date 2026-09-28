Version: 1.3.3-snapshot (xlibs 1.9.0, demonized 20250908)
Changelog: https://github.com/damiansirbu-stalker/AlifeGuard/blob/main/doc/changelog
Health: https://damiansirbu-stalker.github.io/AlifeGuard/health/
JitProfiler: https://damiansirbu-stalker.github.io/AlifeGuard/jitprofiler/
Bugs: https://github.com/damiansirbu-stalker/AlifeGuard/issues
Russian / На русском: https://github.com/damiansirbu-stalker/AlifeGuard/blob/main/doc/readme_ru.txt

My work:
GitHub: https://github.com/orgs/damiansirbu-stalker/repositories
ModDB: https://www.moddb.com/members/damian-sirbu/addons
Nexus: https://www.nexusmods.com/profile/damiansirbu/mods

My contributions:
X-Ray Monolith: https://github.com/themrdemonized/xray-monolith

[ Hero image: alifeguard-hero.gif - the population stays in check ]

! Reset MCM settings to defaults after updating !

Late-game A-Life accumulates too many active entities. AI, physics, and pathfinding all run on the same thread, so performance degrades as the count grows.
Population mods like ZCP and Redone amplify the problem by raising spawn rates. Zombie entities from broken releases, orphaned squad members, and engine memory leaks make it worse over time.

AlifeGuard keeps the world's NPCs, creatures, and items in check, catching what would corrupt a save, break data, or drag performance before it does.
Squads stay valid and smart terrains behave correctly, with no respawn loops. It reduces load without breaking the simulation.

Simple despawners remove individual NPCs without considering squad structure. This deletes entire squads and opens respawn slots.
It leaves smart terrains unable to spawn and interferes with mods that script squads. AlifeGuard works at the squad level and accounts for the engine's spawn bookkeeping.

Systems:
- Online Guard - caps the online population near you: when the count crosses the trigger it culls back to the target, squad by squad.
- Offline Guard - a staggered scan bins offline squads by region and thins the crowded ones, so a hub full of offline squads cannot spike the count when it wakes. Commanders always survive.
- Smart Sanitizer - clamps corrupted respawn counters on smart terrains, the kind that cause save crashes and infinite spawn loops.
- Inventory Guard - bounds what NPCs hoard from looting. Vanilla looting stays on, but a long-lived stalker carries a believable load instead of a trader run.
  Looted items stop drifting toward the engine's ~65000-object cap that crashes long saves.

Mechanisms (how the culling stays safe):
- Squad-aware - thins non-commanders first, commanders last, so squads stay valid in SIMBOARD and no respawn loop starts. Mod-owned squads (AlifePlus, Warfare, Guards Spawner) go last.
- Round-robin - removals spread across factions and mutant types, one per category per round, so no group is thinned disproportionately.
- Hysteresis - separate trigger and target thresholds (default 80 down to 70), so cleanup runs in cycles, not on every small change.
- Frame-spread - entities released one per frame via xslice, so a cleanup spreads over frames and never hitches the game.
- Protection - story squads, traders, named NPCs, companions, task givers, quest/bounty/hostage targets, and scripted squads are never touched.
  Toggle scripted protection off for the most aggressive culling.

Requirements:
Anomaly 1.5.3
Modded exes: themrdemonized 20250908 or newer, or AOEngine v0.55 or newer. The full feature set needs the latest demonized build. A feature that needs a newer one stays inactive on older exes.
xlibs (https://www.moddb.com/mods/stalker-anomaly/addons/xlibs-1001)
MCM

Compatibility:
Depends only on xlibs. Install and uninstall mid-save work. Tested: Anomaly 1.5.3, GAMMA, EFP, Zona, Forgotten Zone.
Disable (conflict, superseded, problematic):
- Grok's Dynamic Despawner, and any other despawn or population-release mod - release the same population AlifeGuard owns, so the two fight over the count.
- Squad Filler - injects offline squads back to size, undoing the cull.
- Cypret's SafeSpawn - force-toggles online switching and sweeps alife every level change, against the release path.
- NPC Stop Looting Dead Bodies, NPC Loot Claim Remade, and any anti-loot mod - hard-block the looting Inventory Guard keeps on and bounds instead.
Coexists:
- AlifePlus, Warfare, Guards Spawner - their scripted squads are protected, so AlifeGuard thins them last and never breaks a mod-owned squad.
It coexists with everything else.

Known issue:
A rare crash on entity release (Perform_reject assertion). This is an engine-level fault in X-Ray inventory parent tracking, present in all population mods, with no script-side fix.

How It's Built:

The code and patterns are original, built on best practices from the best STALKER modders and hands-on reverse-engineering of X-Ray.
The design stays engine-native and minimal, with event-native pub/sub not polling, work spread across frames via deferred queues and rate limiters, and per-level caches replacing world scans.
The raycasting and range math are hand-written and load-tested live, following the engine's own standards and flags.
Where scripting hits an engine limit, the fix is made in X-Ray itself, in the modded exes.
Performance is the first invariant, so every flow stays under 2ms or the build rewrites or drops it, profiled continuously with JitProfiler and hand-tested on unoptimized, single-threaded exes.
Every mod carries OpenTelemetry-style tracing and performance monitoring, spanning world events and every flow, gated by the log level so it costs nothing when off.
Every commit runs the full pipeline locally and in CI, with luacheck, a custom STALKER selene build, and a load test on engine stubs.
Rule layers then check Lua practice, engine truth, conventions, contracts, release, security, and docs.
Every rate, threshold, and toggle is exposed through MCM or LTX with nothing left hard-coded, and it writes no engine values, keeping its state within engine bounds so a save can never corrupt.
It runs on one xlibs rulebook shared across the whole mod family, the same protection, distances, faction logic, and combat reads in every mod.
It depends on no other mod, not even the author's own, and needs only X-Ray and xlibs beneath it.
See the Health and JitProfiler links up top for every test and smoke result, and the mod's real CPU and allocation cost.

Credits:
Altogolik provided support, ideas, and source materials.

Usage and License:
  Modpacks: allowed and encouraged. Keep the readme and license files.
  Addons, patches, integrations: allowed. Credit "AlifeGuard by Damian Sirbu" visibly on your mod page.
  Reproducing the implementation in other software: not allowed, even with credit.
  The full license is in the LICENSE file and on GitHub.

Diagnostics and reporting:
Every release goes through careful engineering and testing, but bugs can still slip through.
To report one, reproduce with debug logging on, and the world log where the mod has one.
First rule this mod out: reproduce with it off, then on. The cleanest test is this mod alone on vanilla and xlibs.
Send the traces on the Anomaly Discord, or file a defect on GitHub with the same information.
Attach xray.log, the mod log, the engine build, the modlist, and the load order.
For deep technical details and mechanisms, check the architecture docs on GitHub.

Tags: alife, performance, engine-native, despawn, population, inventory, squad-aware, offline, offline-cleaning, balanced-culling, protection, safety, save-safe, reverse-engineering
