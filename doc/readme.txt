AlifeGuard: Population control for STALKER Anomaly, by Damian
GitHub: https://github.com/damiansirbu-stalker/AlifeGuard
Changelog: https://github.com/damiansirbu-stalker/AlifeGuard/blob/main/doc/changelog

! Please reset MCM settings to defaults when updating to a new version !

Late-game lag, micro-stutters, FPS drops from A-Life bloat.
Too many online entities choke the X-Ray engine. AlifeGuard keeps the count under a
configurable threshold by despawning the farthest NPCs first. Smart terrains repopulate
naturally. The world stays alive without the CPU cost.

Balanced despawning:
  Farthest-first. All online stalkers and mutants are sorted by distance. The NPC next to you is always the last to go.
  30-second grace period after every level load lets the world settle.
  Single-pass collection via game_objects_iter (native C++ iterator, no Lua table allocation).
  Configurable threshold and interval. Default: 80 max, 30s checks.

Multi-layer protection:
  Every entity goes through 4 independent checks before removal. Any single match keeps it alive.
  Layer 1 - Identity: story characters, squad story IDs, companions, named NPCs (traders, mechanics, leaders, medics, barmen, guides)
  Layer 2 - Task givers: NPCs referenced by active tasks (by NPC ID or squad ID)
  Layer 3 - Bounty/hostage: targets tracked by axr_task_manager
  Layer 4 - Task target squads: assault, bounty, hostage, delivery squads

Performance:
  Frame-spread despawning. Instead of releasing all excess entities in one frame, AlifeGuard
  uses a deferred queue (xslice) to spread the work across frames. Entities are released one
  per frame. 20 excess NPCs = 20 frames to clear. Flat frametimes, no stutter, no freezing.
  Collection uses squared distances, cached engine calls, and weak-key caches for protection
  checks. The mod collects and sorts in one pass, then the queue releases in the background.
  See doc/img/benchmark_despawn_spread.jpg for measured data.

Clean engine release:
  Every removal goes through alife_release_id, one per frame. Releasing multiple entities in
  a single frame is a known cause of X-Ray engine crashes (net_Relcase cascade). One-per-frame
  pacing prevents this entirely. No phantom entities, no orphaned data, no memory leaks.

MCM buttons:
  Show Status     Current entity counts and protection stats via PDA
  Force Cleanup   Immediately cull excess entities
  Delete Common   Remove all respawnable squads (smart terrains repopulate)
  Delete All      Remove ALL squads including story (may break quests)

Debug logging:
  MCM toggle. When enabled, logs every protection check, every release with timing and
  distance, and a full summary line per cycle. Written to alifeguard.log.
  Zero overhead when disabled (all debug calls behind a boolean guard).

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
