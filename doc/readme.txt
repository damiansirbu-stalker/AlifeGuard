AlifeGuard: Population control for STALKER Anomaly, by Damian
GitHub: https://github.com/damiansirbu-stalker/AlifeGuard
Changelog: https://github.com/damiansirbu-stalker/AlifeGuard/blob/main/doc/changelog

Too many online entities kill performance. AlifeGuard keeps the count under a configurable threshold by despawning excess NPCs. Smart terrains repopulate naturally - the world stays alive.

Balanced despawning:
  Farthest-first. All online stalkers and mutants are sorted by distance. The NPC next to you is always the last to go.
  30-second grace period after every level load lets the world settle.
  Single-pass collection via game_objects_iter (native C++ iterator, no Lua table allocation).
  Configurable threshold and interval. Default: 70 max, 30s checks.

Multi-layer protection:
  Every entity goes through 4 independent checks before removal. Any single match keeps it alive.
  Layer 1 - Identity: story characters, squad story IDs, companions, named NPCs (traders, mechanics, leaders, medics, barmen, guides)
  Layer 2 - Task givers: NPCs referenced by active tasks (by NPC ID or squad ID)
  Layer 3 - Bounty/hostage: targets tracked by axr_task_manager
  Layer 4 - Task target squads: assault, bounty, hostage, delivery squads

Performance:
  Single-pass collection via game_objects_iter (native C++ iterator, no Lua table allocation).
  Squared distance comparisons throughout - no sqrt calls.
  All engine API calls (IsStalker, IsMonster, alife_object, alife_release_id) cached as local upvalues - no global lookups on hot paths.
  obj:section() (luabind) only called when debug logging is enabled.
  xcreature.is_unscriptable uses weak-key cache - one luabind call per entity per GC cycle, zero after.
  No per-entity per-frame callbacks. One timer, one pass, done.

Clean engine release:
  Every removal goes through X-Ray's alife_release_id. The entity is removed from every engine table - no zombie phantoms, no orphaned references, no memory leaks.

MCM buttons:
  Show Status     Current entity counts and protection stats via PDA
  Force Cleanup   Immediately cull excess entities
  Delete Common   Remove all respawnable squads (smart terrains repopulate)
  Delete All      Remove ALL squads including story (may break quests)

PDA notifications:
  Immersive messages when cleanup runs. Toggleable via MCM.

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

Usage and License:
  Modpacks: allowed and encouraged. Keep the readme and license files.
  Addons, patches, integrations: allowed. Credit "AlifeGuard by Damian Sirbu" visibly on your mod page.
  Reproducing the implementation in other software: not allowed, even with credit.
  Full license in LICENSE file and on GitHub.
