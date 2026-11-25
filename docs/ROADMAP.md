# Roadmap

HaulInfo follows these phases. Each phase is completed and reviewed before moving to the next.

## Phase 0: Product definition

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

## Phase 9: Testing and optimisation

Test the application, optimise resource usage and rendering performance, and verify no noticeable impact on ETS2.

## Phase 10: HaulInfo evolution and additional features

Add features beyond the initial Android Auto recreation based on the established product direction.

## Phase 11: Release and distribution

Prepare and distribute the first release.