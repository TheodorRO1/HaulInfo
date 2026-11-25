# Product vision

HaulInfo is a companion infotainment and navigation application for Euro Truck Simulator 2.

The initial goal is to bring the latest available Android Auto experience to ETS2 on Windows, recreating its interface and behaviour as closely as technically possible. It should feel familiar to someone who already uses Android Auto in a real car.

The first version focuses on getting the Android Auto experience right, rather than introducing a separate HaulInfo design.

## Design principles

- Recreate the Android Auto interface and user experience as faithfully as possible.
- Use the latest actually released Android Auto version as the reference.
- Do not invent interface elements or features.
- Avoid custom HaulInfo styling during the initial recreation.
- Do not introduce Liquid Glass, custom Material You styling or unrelated visual changes.
- Pay attention to layouts, interactions, animations, navigation and overall behaviour.
- Adapt the experience to ETS2 where necessary without unnecessarily changing its appearance.

Features that have only been announced or are still rolling out are distinguished from features that are actually available.

## Navigation and maps

HaulInfo should eventually have its own navigation system, built using map information extracted automatically from the user's installed ETS2 files.

The intended navigation experience includes:

- 3D navigation
- Roads, highways and interchanges
- Junctions and roundabouts
- Highway signs and road details where the game data supports them
- 3D buildings and landmarks where technically feasible
- Live truck position and movement
- Navigation guidance and route information

Map generation happens automatically. Users do not search for, download or manually install external map datasets.

HaulInfo detects the ETS2 installation, base game, owned DLCs and supported map mods, then generates and maintains its own local map database.

Offline operation is an important requirement.

These are goals, not assumptions about what is technically possible. Their feasibility will be investigated in later phases.

## Initial feature scope

The first version aims to recreate the relevant Android Auto experience:

- Main dashboard
- Navigation interface
- Media interface and controls
- App launcher
- Notifications
- Widgets and information cards
- Settings
- Navigation interactions and transitions
- Responsive layouts

The exact feature list is established during Phase 1, after researching the latest released Android Auto version. No unrelated original features are added at this stage.

## Technical direction

Rust is the primary programming language.

The application prioritises:

- Low resource usage
- Stable performance
- Smooth rendering
- No noticeable impact on ETS2 performance
- Background processing for demanding tasks
- Reliable error handling
- Maintainable and testable code
- Offline functionality where required

Frameworks, rendering technologies, libraries and architectural decisions are evaluated in Phase 3. They are not chosen yet.