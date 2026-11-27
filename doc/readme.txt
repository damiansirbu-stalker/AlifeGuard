Version: 1.3.3-snapshot (xlibs 1.10.1, demonized latest)
Changelog: https://github.com/damiansirbu-stalker/AlifeGuard/blob/main/doc/changelog
Health: https://damiansirbu-stalker.github.io/AlifeGuard/health/
JitProfiler: https://damiansirbu-stalker.github.io/AlifeGuard/jitprofiler/
Bugs: https://github.com/damiansirbu-stalker/AlifeGuard/issues
Russian / На русском: https://github.com/damiansirbu-stalker/AlifeGuard/blob/main/doc/readme_ru.txt

---
Alife mods:
  AlifePlus: https://www.moddb.com/mods/stalker-anomaly/addons/alifeplus-v1-0-01
  AlifeTactics: https://www.moddb.com/mods/stalker-anomaly/addons/alifetactics
  AlifeBalance: https://www.moddb.com/mods/stalker-anomaly/addons/alifebalance
  AlifeGuard: https://www.moddb.com/mods/stalker-anomaly/addons/alifeguard-1001
Diegetic mods:
  DiegeticControl: https://www.moddb.com/mods/stalker-anomaly/addons/diegeticcontrol
  DiegeticAmbience
  DiegeticDread
Tools:
  JitProfiler: https://www.moddb.com/mods/stalker-anomaly/addons/jitprofiler
Libraries:
  xlibs: https://www.moddb.com/mods/stalker-anomaly/addons/xlibs-1001
Engines:
  X-Ray Monolith: https://github.com/themrdemonized/xray-monolith/pulls?q=is%3Apr+author%3Adamiansirbu+is%3Amerged
  OpenXRay: https://github.com/OpenXRay/xray-16/pulls?q=is%3Apr+author%3Adamiansirbu+is%3Amerged
  IX-Ray: https://github.com/ixray-team/ixray-1.6-stcop/commits/develop?author=damiansirbu
Dependents:
  COMPASS: https://github.com/Crimento/COMPASS
  Nemesis Spawn System: https://github.com/SirEndet/Nemesis-Spawn-System
  Word of Mouth: https://www.moddb.com/mods/stalker-anomaly/addons/word-of-mouth
Integrations:
  Warfare (erepb): https://www.moddb.com/mods/stalker-anomaly/addons/warfare-alife-overhaul-new
  Homestead: https://www.moddb.com/mods/stalker-anomaly/addons/homestead-stalker-settlement-builder
  Stealth Overhaul: https://github.com/Alex-leon1594/Stealth_Overhaul_Reworked
  TALKER: https://www.moddb.com/mods/stalker-anomaly/addons/talker-ai-powered-npc-dialogue-beta
  Factions of the Zone: https://github.com/devinhorowitz/factionsofthezone
  Anomaly Radios Extended: https://www.moddb.com/mods/stalker-anomaly/addons/anomaly-radios-extended-full-remaster-152
Collaborations:
  xAGNA: https://www.moddb.com/mods/stalker-anomaly/addons/xagna

[ Hero image: alifeguard-hero.gif - the population stays in check ]

My work is free and open source, ships in most modpacks, and integrates with established projects.
An organized group copies my work, defames, impersonates and review-bombs me across platforms.
You can help with a review, a rating, a proper bug report, or a coffee on Ko-fi: https://ko-fi.com/damiansirbu

Every release is stable, and a new one can add whole systems or move their settings.
After every update, open Development in MCM and reset every setting to its default.

AlifeGuard keeps the A-Life of Anomaly lean through a long game.
Your late game stays fast, and your save stays clear of the 65k crash.

Summary

- AlifeGuard caps the stalkers and mutants online near you and culls the surplus squad by squad.
- When a cull runs, the farthest squads go first, and every squad keeps its commander.
- Crowded parts of the map get thinned too, while they are still offline.
- Every stalker's bag is trimmed to a limit per item type, and the loot can't pile up in the save.
- If a smart terrain's respawn counter breaks, AlifeGuard sets it back to what is really alive.
- Vanilla's offline cleaner, which empties the bags of stalkers on other maps, can be switched off.
- We leave story and quest squads, companions, traders and task givers alone.
- We spread all of it over frames through X-Ray's release path, and nothing goes into your save.

Online Guard

Online Guard keeps the crowd of stalkers and mutants near you under a limit.
Vanilla has no such limit, which is why a late-game base can fill up without end.
- When a cull runs, members go before commanders, the farthest squads go first, and every faction and every mutant type near you then takes a turn in the cut.
  We do this because a squad that keeps its commander also keeps its spawn slot.
- We never cull story characters, traders, named NPCs, companions, task NPCs or scripted squads.
- Your PDA tells you when the Zone has thinned out.

Offline Guard

Offline Guard is how AlifeGuard thins a crowded region of the map before you get there.
In vanilla a hub full of offline squads comes online all at once when you arrive.
- Offline squads are scanned region by region, and a crowded region is thinned one squad per frame.
- Every squad keeps its commander, and we take nothing from protected squads or named NPCs.

Smart Sanitizer

Smart Sanitizer is how AlifeGuard repairs the respawn counter of a smart terrain.
In vanilla a respawn counter that goes negative makes its smart terrain spawn without end.
The next save that holds this counter will crash while writing it out.
- A broken counter is set back to the number of squads still alive, at load and every few minutes during play, but we never touch a counter that is healthy.

Inventory Guard

Inventory Guard is how AlifeGuard keeps what a stalker carries at a believable load.
In vanilla a long-lived stalker loots every body he passes, for nothing trims his bag near you.
- We trim each nearby stalker's bag to a limit per item type, a few items at a time.
- We leave him the scope or silencer that fits his gun, and anything a quest needs.
- Traders, companions and story characters are never trimmed.
- Offline cleaner: vanilla can keep emptying bags of stalkers on other maps, or you can stop it.

Requirements:
Anomaly 1.5.3
Modded exes: themrdemonized 20250908 or newer, or AOEngine v0.5.2 or newer. The full feature set needs the latest demonized build. A feature needing a newer one is inactive or reduced on older exes.
xlibs (https://www.moddb.com/mods/stalker-anomaly/addons/xlibs-1001)
MCM

Update check:
On the first save load of each game launch, the mod asks GitHub during the loading screen whether a newer release is out. It needs xlibs 1.10.1 or newer.
A newer version shows at the foot of its MCM page, and in a PDA message once a day. Check for update in the Development tab asks right away, whenever you want.
GitHub sees your IP address and the names of the mods it checks. Each request carries only the mod's name and the word xlibs, with no save data, settings or player name.

Compatibility:
Depends only on xlibs. Install and uninstall mid-save work. Tested: Anomaly 1.5.3, GAMMA, EFP, Zona, Forgotten Zone.
Drop:
- Grok's Dynamic Despawner - it releases the population AlifeGuard owns and deletes task stalkers.
  Its releases can push a smart terrain's spawn count below zero, which crashes the save.
- Cypret's SafeSpawn - it force-toggles online switching and sweeps alife on every level change.
  A save while it hides stalkers leaves them 200m under the map.
Avoid:
- Any other despawn or population-release mod - it releases the population AlifeGuard owns.
- Squad Filler - injects offline squads back to size, undoing the cull.
- NPC Stop Looting Dead Bodies, and any anti-loot mod - they block NPC looting.
Coexists:
- AlifePlus, Warfare, Guards Spawner - their scripted squads are protected, so AlifeGuard thins them last and never breaks up a squad that another mod owns.
Everything else coexists, as long as it extends X-Ray and Anomaly and never overrides them.

Known issue:
A rare crash on entity release (Perform_reject assertion). This is an engine-level fault in X-Ray inventory parent tracking, present in all population mods, with no script-side fix.

How It's Built:

The code and patterns are original, built on best practices from the best STALKER modders and on hands-on reverse-engineering of X-Ray and the Anomaly scripts.
The design stays engine-native and minimal, with event-native pub/sub not polling, work spread across frames via deferred queues and rate limiters, and per-level caches replacing world scans.
Raycasting and range math are hand-written and load-tested live, on X-Ray's standards and flags.
Where scripting hits an engine limit, the fix is made in X-Ray itself, in the modded exes.
Performance is the first invariant. Every flow stays under 2ms or the build rewrites or drops it, profiled continuously with JitProfiler and hand-tested on unoptimized, single-threaded exes.
Each mod carries OpenTelemetry-style tracing and performance monitoring, spanning world events and every flow, gated by the log level, which skips the tracing work when it is off.
Every commit runs the full pipeline locally and in CI, with luacheck, a custom STALKER selene build, and a load test that runs every script on engine stubs.
Rule layers then check Lua practice, engine truth, conventions, contracts, release, security and docs, and a grade gate fails the build on any code or security finding.
Every rate, threshold and toggle sits in MCM or LTX with nothing hard-coded, and any engine value a mod writes stays within engine bounds, which keeps every save loadable.
It needs only X-Ray and the shared xlibs rulebook beneath it, which gives every mod in the family the same protection, distances, faction logic and combat reads.
It depends on no other mod, the author's other mods included.
See the Health and JitProfiler links up top for every test and smoke result, and for the mod's real CPU and allocation cost, measured in a live game session.

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
First rule this mod out: reproduce with it off, then on.
The cleanest test is this mod alone on vanilla and xlibs.
Send the traces to my Anomaly Discord thread: https://discord.com/channels/456765861953536020/1512539453791670402
Join the server first through the Anomaly Discord invite: https://discord.gg/c4RuJNs
Or file a defect with the same information on GitHub: https://github.com/damiansirbu-stalker/AlifeGuard/issues
Attach xray.log, the mod log, the engine build, the modlist, and the load order.
For deep technical details and mechanisms, check the architecture docs on GitHub.

Tags: alife, performance, fps, crash-prevention, id-limit, stability, engine-native, despawn, population, inventory, squad-aware, offline, offline-cleaning, balanced-culling, protection, safety, save-safe, reverse-engineering
