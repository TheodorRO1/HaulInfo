# HaulInfo

HaulInfo is a companion infotainment and navigation application for Euro Truck Simulator 2 (ETS2) on Windows.

The goal is to bring the Android Auto experience to ETS2, recreating its interface and behaviour as closely as technically possible. It is intended to feel familiar to anyone who already uses Android Auto in a real car.

The first version focuses on getting the Android Auto experience right rather than introducing a separate HaulInfo design.

## Status

This project is in Phase 0 (product definition). No application code has been written yet. Framework, rendering, and architecture decisions have not been made.

## Planned scope

- Android Auto-style dashboard, navigation, media, app launcher, notifications, widgets, and settings
- Navigation built from map information extracted from the user's installed ETS2 files
- Offline operation
- Low resource usage with no noticeable impact on ETS2 performance

## Technology

Primary language: Rust  
Target platform: Windows 10/11 x64  
Target game: Euro Truck Simulator 2

Rendering framework, UI toolkit, and architecture will be selected in Phase 3 after research.

## How it works

These are planned capabilities, not features that already exist.

| Part | What it does |
|------|--------------|
| PC application | Runs HaulInfo and provides the main interface while ETS2 is running. |
| Phone companion app | Connects to the PC application and lets users control supported features from their phone. |
| ETS2 | Provides the truck, game world and information needed for the experience. |
| Navigation system | Uses locally generated map data based on the installed game files and is intended to synchronise with the game's navigation. |

## Roadmap

| Phase | Name | Status |
|-------|------|--------|
| 0 | Product definition | Completed |
| 1 | Android Auto reference research | Completed |
| 2 | ETS2 technical research | Not started |
| 3 | Architecture and technology selection | Not started |
| 4 | Project foundation | Not started |
| 5 | Automatic ETS2 map generation | Not started |
| 6 | Navigation and 3D rendering | Not started |
| 7 | Android Auto interface recreation | Not started |
| 8 | ETS2 integration | Not started |
| 9 | Phone companion app and PC-to-phone communication | Not started |
| 10 | Two-way ETS2 navigation synchronisation | Not started |
| 11 | Testing and optimisation | Not started |
| 12 | HaulInfo evolution and additional features | Not started |
| 13 | Release and distribution | Not started |

The full description of each phase is in docs/ROADMAP.md. The README roadmap and docs/ROADMAP.md are kept in sync.

