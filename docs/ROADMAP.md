# Roadmap

HaulInfo follows these phases. Each phase is completed and reviewed before moving to the next.

## Phase 0: Product definition (Completed)

Define what HaulInfo is, its goals, design principles, and initial scope. Produce the initial documentation.

## Phase 1: Android Auto reference research

Research the latest released Android Auto version. Establish the exact feature list and reference behaviour for the first version.

## Phase 2: ETS2 technical research

Research ETS2 file formats, installation layout, DLC detection, and map mod support. Determine what map information can be extracted automatically.

## Phase 3: Architecture and technology selection

Evaluate and select frameworks, rendering technology, libraries, and architecture. Decisions are based on research from Phases 1 and 2.

## Phase 4: Project foundation

Set up the Rust project structure, build configuration, and core development tooling.

## Phase 5: Automatic ETS2 map generation

Build automatic detection of the ETS2 installation, base game, DLCs, and map mods. Generate and maintain a local map database from extracted game data.

## Phase 6: Navigation and 3D rendering

Implement navigation and 3D map rendering based on the generated map database.

## Phase 7: Android Auto interface recreation

Recreate the Android Auto interface: dashboard, navigation, media, app launcher, notifications, widgets, and settings.

## Phase 8: ETS2 integration

Connect HaulInfo to ETS2 for live truck position and movement data.

## Phase 9: Phone companion app and PC-to-phone communication

Research and implement PC-to-phone communication. Build the phone companion app so users can control supported HaulInfo features from their phone while playing ETS2.

## Phase 10: Two-way ETS2 navigation synchronisation

Synchronise HaulInfo navigation with the game's built-in navigation system in both directions. Selecting a destination in HaulInfo sends it to ETS2, and changing a destination in ETS2 updates HaulInfo.

## Phase 11: Testing and optimisation

Test the application, optimise resource usage and rendering performance, and verify no noticeable impact on ETS2.

## Phase 12: HaulInfo evolution and additional features

Add features beyond the initial Android Auto recreation based on the established product direction.

## Phase 13: Release and distribution

Prepare and distribute the first release.