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

AlifeGuard keeps the online population under control while preserving how A-Life works.
Squads stay valid and smart terrains behave correctly, with no respawn loops. It reduces load without breaking the simulation.

Simple despawners remove individual NPCs without considering squad structure. This deletes entire squads and opens respawn slots.
It leaves smart terrains unable to spawn and interferes with mods that script squads. AlifeGuard works at the squad level and accounts for the engine's spawn bookkeeping.

Squad-aware culling:
Entities are processed as part of their squad, not individually. Non-commander members are thinned first, and commanders are removed only as a last resort.
This keeps squads in SIMBOARD with their already_spawned counters intact, so individual-NPC despawn loops never start.
Squads controlled by other mods (AlifePlus, Warfare, Guards Spawner) are deprioritized. Their members go after unscripted members, and their commanders go last of all.

Round-robin fairness:
Removals are spread across factions and mutant types, one entity per category per round, so no single group is disproportionately affected.

Hysteresis:
Trigger and target thresholds are separate (default: trigger at 80, cull to 70). Cleanup runs in cycles and does not constantly react to small population changes.

Frame-spread release:
Entities are released one per frame via xslice, a cooperative time-slicing helper. 30 excess entities take 30 frames to clear, with no spikes and no freezes.

Protection:
Story squads, traders, named NPCs, companions, task givers, active quest targets, and bounty and hostage targets are never removed.
A squad another script is steering is protected by default in both guards, through the Protect scripted NPCs option.
That covers scripted_target squads: outpost services, chase targets, and mod-spawned guards. Turn it off for the most aggressive culling.
Squad-level checks use a positive-only TTL cache, and a per-member fallback catches named NPCs who are not squad commanders.

Offline Guard:
The online guard only sees entities near you. A hub on another level can hold dozens of offline squads that all come online at once and spike the count.
The Offline Guard runs a staggered scan that bins offline squads into regions by position. It thins any region over the trigger down to the target.
Commanders and protected squads are never removed, so every squad survives. The guard is independent of the online guard, with its own trigger, target, and task protection.
A target of 0 strips a region to lone commanders. Toggle, cull trigger, cull target, task protection, and scan interval live in MCM.

Smart Sanitizer:
A defensive pass clamps corrupted respawn counters on smart terrains.
Negative or non-numeric already_spawned values cause save crashes (u8 write overflow at STATE_Write) and infinite spawn loops (the max greater-than num respawn gate is always true).
The sanitizer runs on actor reinit before the load-time alife burst, then every 300 seconds during play.
It is a Lua table walk over SIMBOARD.smarts and stays sub-millisecond on 50-200 smarts. Toggle and interval live in MCM.

Inventory Guard:
Vanilla NPCs loot corpses, and over a long session this builds up two costs. A long-lived stalker carries a trader run of gear, so killing them becomes a jackpot.
Every looted item is also a permanent alife server object, and the engine tracks at most around 65000 of those. Long saves drift toward the cap, and the game eventually slows, then crashes.

Why not disable NPC looting?
Mods like NPC Stop Looting Dead Bodies and BoltBeGone sidestep the problem by blocking the engine's loot path.
That fixes the symptom, but stalkers no longer loot their kills, which is one of the things that make A-Life feel alive.
Inventory Guard keeps vanilla looting on and bounds the cause. A lightweight scanner walks online stalkers in small batches and releases anything above each category's limit.
Killing a stalker still yields what they actually need to carry, not what they accumulated over 50 game-days.

What you notice:
Long-lived stalkers carry a believable loadout, not a trader-sized hoard. Killing a random stalker yields reasonable loot, not a vendor run.
Saves stay performant across long sessions. Companions, story characters, and named NPCs are never touched.
Traders, mechanics, and medics are matched by role, so even ones spawned at runtime (Warfare and similar) are skipped entirely.
Their stock is left to the trade flow. Vanilla looting still works, and stalkers loot their kills.

Important:
Inventory Guard never spawns items. It releases what NPCs already accumulated above the per-category limits.
Quest items, equipped gear, story_id items, companion gifts, and player-strapped weapons are always preserved.

Example:
An online stalker on Cordon has been alive for 50 game-days. They have picked up 47 medkits, 23 bandages, 18 grenades, and 600 rounds of mismatched ammo.
The scanner's next visit releases 42 medkits (cap 5), 18 bandages (cap 5), 15 grenades (cap 3), and the 600 mismatched rounds (cap 0).
The NPC then walks around with a believable load: a few medkits, a stack of bandages, three grenades, and ammo for the gun they actually carry.

Policy:
Per-category limits live in configs/alifeguard/ag_inventory_policy.ltx (DLTX-overridable).
Defaults cover equipped ammo (in rounds, per tier), grenades, and consumables: medkit, bandage, antirad, stim, pill, food, drink.
Gear coverage includes weapons, outfits, helmets, artefacts, crafting items, and devices.
Quest items, equipped gear, story_id items, companion gifts, and player-strapped weapons are protected by xlibs and never touched.

Inventory Guard moved here from AlifeBalance, where it was called Inventory Balance, and it behaves the same.
If you update both mods, configure it here, because the AlifeBalance tab is gone.

Performance:
Performance comes first, ahead of any feature.
When a feature cannot fit the budget it is reworked or removed, even with an X-Ray engine modification. It is never allowed to slow the game.
Collection is a single pass over a native C++ iterator with cached protection lookups, sub-millisecond for 200 entities.
Release costs 0.05ms, and debug work drops to nothing when the log level is below DEBUG.
The timings are measured on the engine built from the latest source with no multithreading and no optimizations, so they are worst-case.
The optimized multithreaded build you run is always faster.

Mod compatibility:
Most population mods conflict with A-Life mods. AlifeGuard coexists with AlifePlus, Warfare, ZCP, GAMMA, Guards Spawner, and any mod that uses scripted_target.
Scripted squads are protected by default through the Protect scripted NPCs option.
Commanders remain, so squad assignments continue. Spawn counters stay consistent.
Warfare population tracking stays accurate, and AlifePlus cause and consequence chains complete because target squads persist.
Other mods direct the world, and AlifeGuard keeps it performant.

MCM:
Online Guard: population limits, hysteresis buffer, check interval, protection rules (task NPCs, protect scripted, farthest-first, per-squad culling, round-robin), PDA notifications.
Offline Guard: toggle, cull trigger, cull target, task protection, scan interval.
Smart Sanitizer: toggle, interval.
Inventory Guard: toggle, NPCs per frame, rescan cooldown.
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
Requires xlibs. It runs on themrdemonized modded exes 2025.9.10 or newer, or AOEngine v0.55 or newer.
The full feature set needs the latest demonized build, and a feature that needs a newer build stays inactive on older exes.
It does not modify base scripts and uses the standard engine API (alife_release).

Superseded (AlifeGuard does this - drop the other):
- Grok's Dynamic Despawner: same job, but it leaks memory (table.remove during iteration) and drops squad commanders, forcing repeated respawns.
- Anti-loot addons (NPC Stop Looting Dead Bodies, BoltBeGone): Inventory Guard handles NPC looting at the source.

Conflicts (pick one - two governors fight):
- Any other despawn or population-release mod: conflicting releases, repeated respawns, and entity leaks.
- Extended sim-distance or offline-range mods (Living Zone 2000m, Extended Offline, ROAD alife range 666): they expand the population while AlifeGuard contracts it, so the two oscillate.

Affects and coexists:
- A-Life config tweaks (alife.ltx, smart max_population, Redone Alife Performance, x3 perf): an engine-parameter layer with no overlap.
- AlifeBalance: a different axis (respawn acceleration against release work) that composes.
- Weapons Drop on Bodies: it only moves the dying NPC's weapon (corpse against floor) and does not block looting.

Known issue:
A rare crash on entity release (Perform_reject assertion). This is an engine-level fault in X-Ray inventory parent tracking, present in all population mods, with no script-side fix.

Architecture:
- AlifeGuard runs on xlibs, a reverse-engineered API that wraps the X-Ray engine source. Squad lifecycle, protection checks, and spawn bookkeeping were traced through the C++ source.
- Core design patterns were studied from the most accomplished mods in the Anomaly ecosystem.
- No base script edits, no engine patches, runtime callbacks only.
- A two-phase pipeline runs synchronous collection on frame 0, then frame-spread release over frames 1-N. See doc/architecture.md for the full design documentation.

Performance detail:
- Performance was a design constraint from the start. Collection is sub-millisecond for 200 entities, and release costs 0.05ms per frame.
- Cooperative time-slicing (xslice) bounds per-frame release work and prevents frame stutter under heavy cleanup.
- Structured tracing carries trace IDs and per-phase timing, and null object singletons drop debug overhead to nothing when the log level is below DEBUG.

Multi-stage validation pipeline:
- luacheck and selene (static analysis)
- tree-sitter AST analysis and ast-grep structural patterns
- Contract rules (API safety, cross-file dependencies, cyclomatic complexity, coding standards)
- lua54 integration testing with X-Ray engine stubs
- gitleaks (secret scanning)
The full report lives in doc/test-report.log.

FAQ:
Do I need modded exes?
Yes. AlifeGuard needs themrdemonized modded exes (2025.9.10 or newer) or AOEngine (v0.55 or newer). Vanilla Anomaly does not expose the APIs it relies on.

Credits:
Altogolik - support, ideas, source materials

Usage and License:
Modpacks are allowed and encouraged. Keep the readme and license files.
Addons, patches, and integrations are allowed. Credit "AlifeGuard by Damian Sirbu" visibly on your mod page.
Reproducing the implementation in other software is not allowed, even with credit. The full license lives in the LICENSE file and on GitHub.

Reporting issues and suggestions:
Open a report at https://github.com/damiansirbu-stalker/AlifeGuard/issues/new/choose, or ask on the GAMMA, EFP, Anomaly, and Zona Discord servers. Read this readme and the MCM options first.
Include exact repro steps (new game or named save, expected against actual), engine build, modlist, load order, xray.log, and the mod debug log.
With hundreds of mods loaded, only the log shows whether this one was involved.
The debug log is required. Set the MCM log level to DEBUG, reproduce, then set it back to WARN.
DEBUG is not free. It writes a timed line for every evaluation and hitches single-threaded exes.
The millisecond figures include the tracing itself, so treat them as relative.
