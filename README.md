# AlifeGuard: A-Life performance and stability for STALKER Anomaly

A population limiter and alife state repair layer, for performance and save health.
It caps online entities, thins overcrowded regions before you arrive, bounds NPC inventory hoarding, and repairs the respawn counters that corrupt saves. Story characters, companions and quest targets are excluded.

[ModDB](https://www.moddb.com/mods/stalker-anomaly/addons/alifeguard-1001) | [Nexus](https://www.nexusmods.com/stalkeranomaly/mods/104) | [Releases](https://github.com/damiansirbu-stalker/AlifeGuard/releases) | [Bugs, suggestions](https://github.com/damiansirbu-stalker/AlifeGuard/issues)

[![ci](https://github.com/damiansirbu-stalker/AlifeGuard/actions/workflows/ci.yml/badge.svg)](https://github.com/damiansirbu-stalker/AlifeGuard/actions/workflows/ci.yml) [![Project Health](https://img.shields.io/badge/project_health-dashboard-00ced1)](https://damiansirbu-stalker.github.io/AlifeGuard/)

Requires: Anomaly 1.5.3, modded exes (themrdemonized or AOEngine), [xlibs](https://www.moddb.com/mods/stalker-anomaly/addons/xlibs-1001), MCM. Exact versions in [readme.txt](doc/readme.txt).

## My work

- Alife mods: [AlifePlus](https://www.moddb.com/mods/stalker-anomaly/addons/alifeplus-v1-0-01) · [AlifeTactics](https://www.moddb.com/mods/stalker-anomaly/addons/alifetactics) · [AlifeBalance](https://www.moddb.com/mods/stalker-anomaly/addons/alifebalance) · [AlifeGuard](https://www.moddb.com/mods/stalker-anomaly/addons/alifeguard-1001)
- Diegetic mods: [DiegeticControl](https://www.moddb.com/mods/stalker-anomaly/addons/diegeticcontrol) · DiegeticAmbience · DiegeticDread
- Tools: [JitProfiler](https://www.moddb.com/mods/stalker-anomaly/addons/jitprofiler)
- Libraries: [xlibs](https://www.moddb.com/mods/stalker-anomaly/addons/xlibs-1001)
- Engines: [X-Ray Monolith](https://github.com/themrdemonized/xray-monolith/pulls?q=is%3Apr+author%3Adamiansirbu+is%3Amerged) · [OpenXRay](https://github.com/OpenXRay/xray-16/pulls?q=is%3Apr+author%3Adamiansirbu+is%3Amerged)
- Integrations: [Word of Mouth](https://github.com/joshcoppola/word_of_mouth) · [Warfare (erepb)](https://www.moddb.com/mods/stalker-anomaly/addons/warfare-alife-overhaul-new) · [Stealth Overhaul](https://github.com/Alex-leon1594/Stealth_Overhaul_Reworked) · [COMPASS](https://github.com/Crimento/COMPASS)
- Collaborations: [xAGNA](https://www.moddb.com/mods/stalker-anomaly/addons/xagna)

## Documentation

- [readme.txt](doc/readme.txt) - full description, features, performance
- [changelog](doc/changelog) - version history
- [architecture.md](doc/architecture.md) - protection layers, performance, engine integration

## License

PolyForm Perimeter License. See [LICENSE](LICENSE).
