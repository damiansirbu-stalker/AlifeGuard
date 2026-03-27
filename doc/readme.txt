AlifeGuard: Population control for STALKER Anomaly, by Damian
Latest: 1.0.7 (xlibs 1.2.1)
GitHub: https://github.com/damiansirbu-stalker/AlifeGuard

Too many online entities kill performance. AlifeGuard maintains a target count by releasing excess NPCs back to offline simulation. They continue existing in A-Life, just not rendered.

Every removal goes through the correct X-Ray release chain: alife_release_id triggers squad:remove_npc, which cascades through smart terrain unregistration, squad member cleanup, and the server_entity_on_unregister callback. The entity is gone from every engine table that knew about it. No zombie phantoms, no orphaned references, no memory leaks.

Removal starts with the farthest entities. All online stalkers and mutants are collected, sorted by distance to the player, and released farthest-first until under the target. The NPC standing next to you is always the last to go.

Protected entities are never removed. Traders, mechanics, leaders, medics, barmen, guides, story characters, companions, companion squads, NPCs with active tasks, entities with story IDs. Multiple checks run on every entity before removal - any one is enough to keep it.

A 30-second grace period after every level load lets the world settle before any cleanup runs.

Four MCM buttons give direct control: show current entity stats, force an immediate cleanup, delete all common (respawnable) squads, or nuclear option - delete all squads including story ones. Smart terrains repopulate naturally after a common squad delete.

Features:

Online Guard:
  Monitors online entity count on a configurable interval
  Releases farthest entities first when over threshold
  30-second grace period after level load
  Distance-based or random removal (configurable)

Protection:
  Traders, mechanics, leaders, medics, barmen, guides - never removed
  Story characters and quest givers - never removed
  Companions and companion squads - never removed
  NPCs with active tasks - never removed (configurable)

MCM Buttons:
  Show Status         Display current entity counts and protection stats via PDA
  Force Cleanup       Immediately cull excess entities
  Delete Common       Remove all respawnable squads (smart terrains repopulate)
  Delete All          Remove ALL squads including story (may break quests)

Notifications:
  Immersive PDA messages when cleanup runs
  Debug logging to alifeguard.log (optional)

Requirements:
Anomaly 1.5.3
Modded exes
xlibs (https://www.moddb.com/mods/stalker-anomaly/addons/xlibs-1001)
MCM

Install (MO2):
1. Install xlibs
2. Install AlifeGuard
3. Load order does not matter
4. Configure via MCM

Uninstall (MO2):
Disable or remove in MO2.

Configuration:
All settings in MCM under AlifeGuard. The defaults work well, adjust after playing.

Compatibility:
Works with AlifePlus, ZCP, Warfare, GAMMA, and any other A-Life or warfare mod. Does not modify base scripts. Only releases entities through the standard engine API.

Known Issues:
Extremely rare crash on entity release (Perform_reject assertion).
This is an engine bug in X-Ray's inventory parent tracking - the same code path used by all similar mods: Grok Dynamic Despawner, Night Mutants (xcvb), Phantoms (xcvb), Guards Spawner (xcvb), Dynamic Anomalies (Demonized), Boomsticks and Sharpsticks despawn scripts (Mich).
No script-side fix exists.

Development:
Written against X-Ray Monolith engine source, Demonized exes source code, and Anomaly 1.5.3 unpacked gamedata.
Code patterns and engine usage validated against established work by reputable GAMMA modders (Demonized, Vintar0, RavenAscendant, xcvb).
The code is validated in real time by a multi-stage pipeline: luacheck, selene, tree-sitter AST analysis, contract rules, cross-file dependency resolution, cyclomatic complexity analysis, crash and vulnerability pattern detection, lua54 integration testing with X-Ray engine stubs, gitleaks secret scanning.
Full report in doc/test-report.log.

Credits:
Stalker_Boss - Russian translation

Versions:

1.0.7
  Changed: xlibs dependency updated to 1.2.1

1.0.6
  Changed: default online guard threshold from 40 to 70
  Changed: MCM config access performance optimization
  Added: MCM reset button support
  Added: MCM label cleanup
  Fixed: smaller performance optimizations

1.0.5
  Fixed: dependency gate uses exact version match instead of string comparison

1.0.4
  Task giver protection fix and debug tracing.
  Fixed: sim task givers unprotected (task_giver_id is squad ID, not NPC ID)
  Added: debug log traces for all protection decisions and released entity sections

1.0.3
  Dependency gate and Russian translation.
  Added: dependency gate - clear error message if xlibs is missing or outdated
  Added: Russian translation (Stalker_Boss)

1.0.2
  Fixed: bounty and hostage task targets protected from release

1.0.1
  Fixed console log spam during entity iteration and release.
  Fixed: alive() and release callstack spam on non-creature objects
  Fixed: missing check_interval default

1.0.0
  First release. Online entity limiter with MCM configuration.
  Added: online guard with distance-based removal
  Added: entity protection (traders, companions, story, tasks)
  Added: MCM buttons (status, cleanup, delete common, delete all)
  Added: immersive PDA notifications
  Added: debug logging
