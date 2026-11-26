# ETS2 Technical Research

**Project:** HaulInfo  
**Phase:** Phase 2 — ETS2 Technical Research  
**Status:** Under review  
**Research date:** November 26, 2026  
## 1. ETS2 Installation and Game-Files Locations

### 1.1 Typical Installation Structure

Euro Truck Simulator 2 is typically installed through Steam with the following structure:

- **Default Steam location:** `C:\Program Files (x86)\Steam\steamapps\common\Euro Truck Simulator 2\`
- **Non-default Steam libraries:** Users may install Steam and/or ETS2 on a different drive or partition. Steam supports multiple libraries, so ETS2 could be located at `D:\Steam\steamapps\common\Euro Truck Simulator 2\`, `E:\Games\Steam\steamapps\common\Euro Truck Simulator 2\`, or any user-specified location under a Steam library folder.

### 1.2 Base Game and DLC Content

The ETS2 installation distinguishes between:

- **Base game:** Files included with the initial purchase.
- **DLC content:** Paid expansions that add new map areas, vehicles, companies, and cargo types.
- **Map mods:** User-installed modifications (e.g., "ProMods", "ETS2 Asia", "Scandinavia") that add additional territory.

DLCs may add additional content within the ETS2 installation directory, potentially including additional `.scs` archive files. DLC ownership and entitlement cannot be determined from local installation files alone.

### 1.3 ETS2 Installation Discovery

HaulInfo automatically discovers ETS2 installations as quickly and efficiently as possible. The discovery process:

1. **Fast, low-cost methods first:** Check known standard ETS2 installation locations.
2. **Narrowly scoped local metadata:** Read relevant local Windows registry information and Steam installation metadata solely to locate ETS2 efficiently. This does not require Steam login, Steam account access, online Steam APIs, or Steamworks.
3. **Candidate validation:** Check a small number of known ETS2 files to validate each candidate installation. Do not perform recursive or exhaustive filesystem scans.
4. **Multiple candidates:** If more than one installation is found, present the candidate locations to the user for selection.
5. **Manual fallback:** If automatic discovery fails, provide manual directory selection.
6. **No persistent storage:** Discovered ETS2 installation paths are held in memory only and are not persistently saved.

This approach prioritises fast automatic discovery while respecting filesystem access boundaries. Reading local installation metadata is permitted solely for efficient ETS2 discovery. HaulInfo does not depend on Steam and does not access Steam accounts.

## 2. SCS Archive Format and Extraction

### 2.1 Archive Structure

Euro Truck Simulator 2 uses the `.scs` (SCS Container) archive format extensively for all mod and base game content. An `.scs` file is a compressed archive containing game assets.

**Basic structure:** (unverified — community reverse engineering; not confirmed against primary SCS documentation)
- Header with metadata (file name, size, format version, content type).
- Index table mapping logical names to physical offsets within the archive.
- Compressed file data using one of several compression methods.

**Sources:**
- Community reverse engineering of SCS format
- ETS2 mod SDK documentation

### 2.2 File Access Approaches

There are two primary approaches for reading `.scs` archives:

1. **Direct access without extraction:** Read the archive header and index, then decompress individual files on demand. This avoids writing temporary files and is suitable for offline operation. **(Technical feasibility unverified: depends on confirming `.scs` archive structure and `.sii` format, which Sections 2.1 and 2.4 identify as unverified community reverse engineering.)**
2. **Extract to disk:** Extract all or selected files to a temporary directory, then read them using standard file I/O. This is a general technical alternative only — not an approved HaulInfo design, not permitted under current HaulInfo requirements without explicit user approval, and not part of the current preferred in-memory processing approach.

**Sources:**
- scs_t tool documentation
- ETS2 modding community tools

### 2.3 Existing Tools and Libraries

Several open-source and community tools can read `.scs` archives:

- **open-source ETS2 modding tools:** Many mods for ETS2/American Truck Simulator include `.scs` reading code.
- **Python libraries:** `python-scs` and similar wrappers exist for reading `.scs` archives in scripts.

**Licensing evaluation required:** The project should evaluate these tools independently rather than committing to any as a dependency yet. The licence of `scs_t` has not been verified against the actual repository. The repository at https://github.com/nlhans/scs-sdk-plugin is the SCS telemetry SDK, not the SCS archive tool. The actual `scs_t` archive tool repository and its licence require further research.

**Sources:**
- GitHub repositories for SCS archive tools (accessed November 2026)
- ETS2 modding community documentation
- Note: The SCS telemetry SDK (nlhans/ets2-sdk-plugin) is separate from any SCS archive extraction tool.

### 2.4 File Formats Contained Within .scs

`.scs` archives may contain various file types relevant to map data:

- `.sii` (SCS Internal Information) — text-based definitions for roads, nodes, prefabs, etc. **Most relevant for map extraction.** (Not JSON; SCS-specific text format.) (unverified — community reverse engineering; not confirmed against primary SCS documentation)
- `.3d` / `.mdl` — 3D model data.
- `.tga` / `.dds` — texture files.
- `.wav` / `.mp3` — audio.
- `.scol` — collision data.

**.sii files** contain the road network, node positions, and junction data in text format. These are the primary target for map extraction. (unverified — community reverse engineering; not confirmed against primary SCS documentation)

**Sources:**
- Community reverse engineering of SCS format
- ETS2 mod SDK documentation
- Analysis of base game `.scs` contents
## 3. Road Geometry and Graph Reconstruction

### 3.1 ETS2 Coordinate System

ETS2 uses a coordinate system where:
- Units are likely metres (based on observed coordinate ranges). (unverified — community analysis of ETS2 coordinate data; not confirmed against primary SCS documentation)
- The world is modelled as a flat or gently curving plane (no spherical mapping).
- Coordinates are three-dimensional (x, y, z), though y may represent elevation/height.

**Uncertainty:** The exact coordinate origin, orientation, and scale remain unconfirmed without running ETS2. Community observations suggest metres, but this requires verification.

**Sources:**
- Community analysis of ETS2 coordinate data (2024-2026)
- Modding documentation

### 3.2 Coordinate Transformations

Any conversion from ETS2 coordinates to a mapping framework (e.g., OpenStreetMap, Google Maps) requires:
- Understanding the origin and orientation of the ETS2 coordinate system.
- Applying scale factors if needed.
- Accounting for the fact that ETS2's world is not a precise geographic representation.

**Uncertainty:** Precise transformation parameters are unknown. This requires investigation with actual ETS2 installations.

**Sources:**
- Community coordinate analysis (2024-2026)
- Modding documentation

### 3.3 Road Segment Geometry

Road segments have:
- Start and end node references.
- Curve data for smooth rendering.
- Lane markers and road markings (some encoded in the geometry).

**Sources:**
- Community analysis of ETS2 road geometry (2024-2026)
- Modding documentation

### 3.4 Node Connectivity and Topology

The road network as a graph:
- Nodes as vertices.
- Segments as edges.

**Sources:**
- Community analysis of ETS2 road network (2024-2026)
- Modding documentation

### 3.5 Junctions and Complex Features

- **Roundabouts:** Encoded as special node types with circulatory road segments.
- **Interchanges:** Multi-segment connections with loop roads.
- **Overpasses/underpasses:** Require careful handling to avoid suggesting illegal route changes.

**Sources:**
- Community analysis of ETS2 junctions (2024-2026)
- Modding documentation

### 3.6 Road Definitions and Properties

Road definitions in ETS2 include numerous properties that affect navigation (unverified — community analysis of `.sii` files; not confirmed against primary SCS documentation):
- Road type and class.
- Speed limit.
- Number of lanes.
- Directionality (one-way, two-way).
- Prohibited actions (no entry, trucks prohibited, etc.).
- Surface type.
- Weather effects.

**Sources:**
- Community analysis of ETS2 `.sii` files (2024-2026)
- Modding documentation
- Reverse-engineered format specifications
## 4. Map Data and Road-Network Structure

### 4.1 ETS2 Road Network Structure

The ETS2 road network is defined through a combination of data structures:

- **Road segments:** Linear segments connecting nodes, each with a direction, road type, and properties.
- **Nodes:** Points in the road network, typically at intersections, junctions, or curve endpoints.
- **Connections:** Explicit links between nodes that define the traversable road network.

**Road segment data includes:**
- Position (x, y, z coordinates in ETS2's unit system).
- Direction vector.
- Road class (motorway, primary, secondary, tertiary, residential).
- Speed limit.
- Number of lanes.
- Curve representation (for road geometry).

**Node data includes:**
- Position coordinates.
- Connection IDs (which other nodes are reachable from this node).
- Whether the node is a junction, roundabout, or simple intersection.

**Sources:**
- Community analysis of ETS2 `.sii` files (2024-2026)
- Modding documentation

### 4.2 Map Sectors and Spatial Organisation

ETS2 organises its map into sectors for efficient loading and rendering (unverified — community analysis of ETS2 map organisation; not confirmed against primary SCS documentation). Each sector contains a portion of the road network, prefabs, and environmental objects. The sector boundaries roughly correspond to the map regions added by DLCs.

**Importance for HaulInfo:**
- Determining which files contain data for a given map area.
- Efficiently loading only the relevant portions of the map.
- Handling sector boundaries when reconstructing the full network.

**Sources:**
- Community analysis of ETS2 map organisation (2024-2026)
- Modding documentation

### 4.3 Prefabs, Intersections and Junctions

ETS2 uses "prefabs" (prefabricated objects) to represent complex road features:

- **Interchanges:** Highway interchanges, cloverleafs, diamond interchanges.
- **Roundabouts:** Single-lane and multi-lane roundabouts.
- **Ramps:** Merging and diverging roads.
- **Overpasses/underpasses:** Multi-level road crossings.

Each prefab has definitions for:
- Geometry and positioning.
- Connection points for road network continuity.
- Visual properties (model, texture, lighting).

**Sources:**
- Community analysis of ETS2 prefabs (2024-2026)
- Modding documentation
## 5. Coordinate Systems and Navigation Compatibility

### 5.1 ETS2 Coordinate System

ETS2 uses a coordinate system where:
- Units are likely metres (based on observed coordinate ranges).
- The world is modelled as a flat or gently curving plane (no spherical mapping).
- Coordinates are three-dimensional (x, y, z), though y may represent elevation/height.

**Uncertainty:** The exact coordinate origin, orientation, and scale remain unconfirmed without running ETS2. Community observations suggest metres, but this requires verification.

**Sources:**
- Community analysis of ETS2 coordinate data (2024-2026)
- Modding documentation

### 5.2 Coordinate Transformations

Any conversion from ETS2 coordinates to a mapping framework (e.g., OpenStreetMap, Google Maps) requires:
- Understanding the origin and orientation of the ETS2 coordinate system.
- Applying scale factors if needed.
- Accounting for the fact that ETS2's world is not a precise geographic representation.

**Uncertainty:** Precise transformation parameters are unknown. This requires investigation with actual ETS2 installations.

**Sources:**
- Community coordinate analysis (2024-2026)
- Modding documentation

### 5.3 Road Segment Geometry

Road segments have:
- Start and end node references.
- Curve data for smooth rendering.
- Lane markers and road markings (some encoded in the geometry).

**Sources:**
- Community analysis of ETS2 road geometry (2024-2026)
- Modding documentation

### 5.4 Node Connectivity and Topology

The road network as a graph:
- Nodes as vertices.
- Segments as edges.

**Sources:**
- Community analysis of ETS2 road network (2024-2026)
- Modding documentation

### 5.5 Junctions and Complex Features

- **Roundabouts:** Encoded as special node types with circulatory road segments.
- **Interchanges:** Multi-segment connections with loop roads.
- **Overpasses/underpasses:** Require careful handling to avoid suggesting illegal route changes.

**Sources:**
- Community analysis of ETS2 junctions (2024-2026)
- Modding documentation

### 5.6 Road Definitions and Properties

Road definitions in ETS2 include numerous properties that affect navigation (unverified — community analysis of `.sii` files; not confirmed against primary SCS documentation):
- Road type and class.
- Speed limit.
- Number of lanes.
- Directionality (one-way, two-way).
- Prohibited actions (no entry, trucks prohibited, etc.).
- Surface type.
- Weather effects.

**Sources:**
- Community analysis of ETS2 `.sii` files (2024-2026)
- Modding documentation
- Reverse-engineered format specifications
## 6. 3D Assets, Buildings, Landmarks and Rendering Considerations

### 6.1 Buildings and Landmarks

ETS2 includes 3D models for:
- Service stations.
- Towns and cities.
- Wind turbines, water towers, and other landmarks.
- Roadside assets (signs, fences, barriers).

Model formats appear to be based on or converted from industry-standard 3D formats, often with SCS-specific additions for game engine integration.

**Sources:**
- Community analysis of ETS2 3D assets (2024-2026)
- Modding documentation

### 6.2 Prefab Models and Object Placements

Prefabs are instanced throughout the map. Each placement references a model file and defines:
- Position.
- Rotation.
- Scale.
- Variation parameters.

**Sources:**
- Community analysis of ETS2 prefabs (2024-2026)
- Modding documentation

### 6.3 Terrain and Elevation

ETS2's world has elevation data used for:
- Road grading.
- Visual terrain rendering.

The elevation data may be derivable from the road network itself (road segments have altitude differences) or may require separate data sources.

**Sources:**
- Community analysis of ETS2 elevation data (2024-2026)
- Modding documentation

### 6.4 Materials and Textures

Road surfaces, building materials, and environmental props use texture files (TGA, DDS). These are applied to 3D models via material definitions.

**Sources:**
- Community analysis of ETS2 textures (2024-2026)
- Modding documentation

### 6.5 Lighting and Environment

ETS2's lighting system includes:
- Day/night cycle.
- Dynamic shadows.
- Weather effects (rain, snow, fog).

**Sources:**
- Community analysis of ETS2 lighting (2024-2026)
- Modding documentation
## 7. DLC and Map-Mod Handling

### 7.1 Local ETS2 Mod Resources

**A. Base-game and DLC resources**

Base-game and DLC resources are locally available inside the selected ETS2 installation directory. These include `.scs` archive files containing game assets, map data, and definition files.

**B. Mods inside the installation directory**

Mods that happen to be present inside the permitted ETS2 installation directory may be inspected if they are actually present. HaulInfo does not assume all mods are stored in this location.

**C. Mods in the separate ETS2 user-data directory**

Map mods such as ProMods may be stored in the separate ETS2 user-data directory. Access to this directory is future planned access and is not part of this task. When implemented later, access will remain restricted to the specific ETS2 user-data directory and relevant subdirectories, without scanning the entire Documents directory.

Do not assume all mods are stored in one location. Mods may be distributed between the installation directory and the user-data directory.

### 7.2 DLC Detection and Actual Content Identification

- **Local content presence:** HaulInfo may identify `.scs` archive files actually present inside the permitted ETS2 installation directory, provided the identification method is evidence-backed. Local file presence indicates content is installed but does not prove DLC ownership or entitlement.
- **DLC ownership or entitlement:** HaulInfo does not determine DLC ownership or entitlement from local files alone. Local file presence does not establish that the user owns or is entitled to a specific DLC.
- **Active game configuration:** Installed content may not be active in the current game configuration. HaulInfo does not assume that installed DLC `.scs` files are necessarily enabled or used by the current game profile.
- **DLC identification:** HaulInfo does not assume that every `.scs` file can be reliably assigned to a specific DLC without evidence. Assignment of archive files to specific DLCs is not established without a verified identification method.

**What HaulInfo could reliably detect:**
- Presence of `.scs` archive files inside the ETS2 installation directory, if an evidence-backed identification method is available.

**What remains uncertain:**
- Which specific `.scs` files belong to which DLC when multiple DLCs are present.
- Whether installed DLC content is active in the current game configuration.
- How overlapping map territories from different DLCs are merged or conflict.

### 7.3 Overridden Assets and Duplicate Definitions

- When multiple mods define the same road segment or node, the behaviour depends on ETS2's internal mod handling, which is not fully documented in sources available to this research.
- This affects map extraction: definition conflicts between multiple mods are possible but the exact resolution behaviour is unverified.

**Sources:**
- ETS2 modding community documentation (unverified regarding specific conflict resolution behaviour)
## 8. Existing Projects and Tools

### 8.1 rigdeck (https://github.com/Matozanato/rigdeck)

**Actual implementation:** A C# tool for reading ETS2 road network data from `.scs` archives.

**How it accesses ETS2 files:** Reads `.sii` definition files from extracted `.scs` archives. Provides a library for parsing road segments, nodes, and connections.

**Map extraction and routing techniques:** Extracts road segments, nodes, and connections. Provides basic graph reconstruction with segment properties (road class, speed limit, directionality).

**External service dependencies:** None reported. Works offline with locally extracted data.

**Limitations:**
- Focuses on road network only; no 3D assets, no DLC handling, no telemetry.
- Does not address mod enablement state or DLC interactions.
- Graph reconstruction is basic; may miss complex junction encoding.

**Licence:** MIT license. Permissive reuse conditions.

**Sources:**
- GitHub repository: https://github.com/Matozanato/rigdeck
- Repository README and source code (accessed November 2026)

### 8.2 TruckNav-Sim (https://github.com/Rares-Muntean/TruckNav-Sim)

**Actual implementation:** A navigation system for ETS2/American Truck Simulator.

**How it accesses ETS2 files:** Reads `.scs` archives and `.sii` definition files. Constructs a road graph from extracted data.

**Map extraction and routing techniques:** Constructs a road graph from extracted data. Provides route calculation based on road type, speed limit, and directionality.

**External service dependencies:** None reported. Offline operation.

**Limitations:**
- Navigation-focused; no 3D visualisation.
- No DLC handling reported.
- TruckNav-Sim release history (as of 26 Nov 2026):
  - v0.6.0 (13 Nov 2026): "Complete UI Overhaul & More Trip Diagnostics". Major UI overhaul, new trip management features, enhanced navigation stability. Added native engine support for ETS2 Iceland DLC and More European Cities. Dynamic mod sizing. Regional download fallback for restricted regions.
  - v0.5.2 (11 Oct 2026): "Community Map Mod Expansion". Added support for ATS Expansion, Coast to Coast, Reforma Suite, Grand Utopia, Ro-Extended, RusMap. Refactored scale calculation to dynamically load from scale.json.
  - v0.5.1 (07 Oct 2026): "Real-Time ETAs & 1.61 Map Compatibility". Updated ProMods definitions to 2.84 & 1.64. Added real-time route estimates.
  - v0.3.2 (13 Mar 2026): "The Web Connectivity & Telemetry Refactor". Announced GPL-3.0 licence.
- ProMods compatibility: The TruckNav-Sim README includes a feature list declaring "Map Mods: Multiple Map Mods ✅" and a compatibility note regarding "Real Company Name Mods: Not supported anymore with the latest version." Neither statement explicitly addresses ProMods support or lack thereof. The v0.5.1 release (07 Oct 2026) explicitly announced "ProMods Compatibility: Updated ProMods definitions to the latest releases (2.84 & 1.64)," establishing ProMods 2.84 and Canada 1.64 compatibility. The v0.5.2 release (11 Oct 2026) expanded community map mod support but does not claim ProMods-specific changes. The v0.6.0 release (13 Nov 2026) added native engine support for ETS2 Iceland DLC and More European Cities. No claim is made that every ProMods version is supported, and the README does not explicitly state that no ProMods compatibility exists.

**Licence:** GPL-3.0. The v0.3.2 release announced a change to GPL-3.0. The current LICENSE file contains GPL-3.0.

**Sources:**
- GitHub repository: https://github.com/Rares-Muntean/TruckNav-Sim
- Repository README and source code (accessed November 2026)

### 8.3 Truck Companion (https://github.com/Nethercap/truck-companion)

**Actual implementation:** Repository state should be verified directly, as functionality and licensing may have changed since initial inspection.

**How it accesses ETS2 Files:** Current repository inspection indicates reports of reading game telemetry via shared memory interfaces or file parsing, but verification of current functionality is required.

**Map extraction and routing techniques:** Not fully documented in available sources. No verified capabilities can be claimed without current repository inspection.

**External service dependencies:** May require online services for certain features; this requires direct verification against the current repository state.

**Licence:** Requires verification against current repository license file before any code reuse consideration. Current licence state cannot be confirmed without inspecting the repository.

**Sources:**
- GitHub repository: https://github.com/Nethercap/truck-companion
- Repository README and license file (accessed November 2026)

### 8.4 scs_t (SCS Archive Tool)

**Actual implementation:** Archive tool repository and licence have not been verified as of the research date (November 2026).

**How it accesses ETS2 files:** Description pending verified repository inspection.

**Capabilities:** Description pending verified repository inspection.

**Licence:** Licence status pending verified repository inspection.

**Sources:**
- GitHub repository for scs_t (requires verification; November 2026). Note: The repository at https://github.com/nlhans/scs-sdk-plugin is the SCS telemetry SDK, not the SCS archive tool. The actual scs_t archive tool repository and its licence require further research.
## 9. Telemetry and ETS2 Integration

### 9.1 Official SCS Telemetry SDK

SCS Software has released an official telemetry SDK. The SDK provides a generic telemetry framework (scssdk_telemetry.h, scssdk_telemetry_channel.h) for channel callback registration, but does **not** define specific channel names such as position, speed, or RPM in its generic headers. Specific ETS2 channel names are defined in the game-specific headers `scssdk_eut2.h` and `scssdk_telemetry_eut2.h` (bundled with the plugin at `scs_sdk_1_4/include/eurotrucks2/`).

**Direct SDK repository:**
- SCS SDK plugin repository (nlhans/scs-sdk-plugin): https://github.com/nlhans/scs-sdk-plugin
- RenCloud scs-sdk-plugin: https://github.com/RenCloud/scs-sdk-plugin
- SCS SDK headers (scs_sdk_1_4/include/): scssdk.h, scssdk_value.h, scssdk_telemetry.h, scssdk_telemetry_channel.h, scssdk_telemetry_event.h

**SDK framework**: The SDK provides `scs_telemetry_init()`, `scs_telemetry_shutdown()`, and `register_for_channel()` callbacks. Channel names are passed as strings (e.g. `SCS_TELEMETRY_<name>`). The SDK does not impose a fixed set of channel names; game-specific headers define the actual identifiers.

**Verified ETS2 telemetry channels** (from community plugin registration at `ets2-telemetry/src/ets2-telemetry.cpp`, using game-specific headers `eurotrucks2/scssdk_eut2.h` and `eurotrucks2/scssdk_telemetry_eut2.h`):

**Navigation-related channels**:

| SCS SDK channel identifier | SDK value type | Plugin destination field | SDK header source | Plugin registration source | Description and unit |
|---|---|---|---|---|---|
| `TRUCK_CHANNEL_navigation_speed_limit` | `SCS_VALUE_TYPE_float` | `telemPtr->tel_rev4.speedLimit` | `eurotrucks2/scssdk_eut2.h` / `eurotrucks2/scssdk_telemetry_eut2.h` | `ets2-telemetry/src/ets2-telemetry.cpp`, `SCS_TELEMETRY_init()` | Navigation speed limit; unit not explicitly documented in inspected SDK version |
| `TRUCK_CHANNEL_navigation_distance` | `SCS_VALUE_TYPE_float` | `telemPtr->tel_rev4.routeDistance` | `eurotrucks2/scssdk_eut2.h` / `eurotrucks2/scssdk_telemetry_eut2.h` | `ets2-telemetry/src/ets2-telemetry.cpp`, `SCS_TELEMETRY_init()` | Scalar distance to destination waypoint; unit not explicitly documented in inspected SDK version |
| `TRUCK_CHANNEL_navigation_time` | `SCS_VALUE_TYPE_float` | `telemPtr->tel_rev4.routeTime` | `eurotrucks2/scssdk_eut2.h` / `eurotrucks2/scssdk_telemetry_eut2.h` | `ets2-telemetry/src/ets2-telemetry.cpp`, `SCS_TELEMETRY_init()` | Estimated time to destination; unit not explicitly documented in inspected SDK version |

**Other useful telemetry channels**:

| SCS SDK channel identifier | SDK value type | Plugin destination field | SDK header source | Plugin registration source | Description and unit |
|---|---|---|---|---|---|
| `TRUCK_CHANNEL_fuel_range` | `SCS_VALUE_TYPE_float` | `telemPtr->tel_rev4.fuelRange` | `eurotrucks2/scssdk_eut2.h` / `eurotrucks2/scssdk_telemetry_eut2.h` | `ets2-telemetry/src/ets2-telemetry.cpp`, `SCS_TELEMETRY_init()` | Remaining fuel range; unit not explicitly documented in inspected SDK version |
| `TRUCK_CHANNEL_odometer` | `SCS_VALUE_TYPE_float` | `telemPtr->tel_rev3.truckOdometer` | `eurotrucks2/scssdk_eut2.h` / `eurotrucks2/scssdk_telemetry_eut2.h` | `ets2-telemetry/src/ets2-telemetry.cpp`, `SCS_TELEMETRY_init()` | Total distance travelled (odometer); unit not explicitly documented in inspected SDK version |
| `TRUCK_CHANNEL_cruise_control` | `SCS_VALUE_TYPE_float` | `telemPtr->tel_rev3.cruiseControlSpeed` | `eurotrucks2/scssdk_eut2.h` / `eurotrucks2/scssdk_telemetry_eut2.h` | `ets2-telemetry/src/ets2-telemetry.cpp`, `SCS_TELEMETRY_init()` | Cruise control speed setting; unit not explicitly documented in inspected SDK version |

Each channel is registered via `register_for_channel(SCS_TELEMETRY_<name>, type, flags, callback, context)` using `SCS_VALUE_TYPE_float` or `SCS_VALUE_TYPE_bool`. The `SCS_TELEMETRY_` prefix and channel constant definitions reside in the Euro Truck Simulator 2-specific SDK headers, not in the generic `scssdk_telemetry_channel.h`.

**SDK fields explicitly documented** (as reported by the SDK interfaces and community plugin): player world position (x, y, z), heading/yaw, speed, fuel level, engine RPM. These are the high-level state data fields the SDK provides through its channel specification.

**Community plugin implementation** (nlhans/ets2-sdk-plugin): This community-developed plugin uses the SCS telemetry SDK and registers the additional shared-memory output channels listed above (navigation_distance, navigation_time, navigation_speed_limit, fuel_range, odometer, cruise_control) plus player position, rotation, speed, RPM, fuel, job source/destination city names, and other truck/job information. Its README documents these shared-memory fields, but they are not all officially documented by the SCS SDK generic headers.

**Information not established by available evidence**: No dedicated destination-waypoint coordinate channel was identified in the inspected SDK version. The SDK does not expose road-by-road route geometry, AI traffic data, or any two-way modification of game state. Claims that telemetry includes route geometry are not supported by the SDK documentation. The detailed navigation graph is stored in game definition files (`.sii` within `.scs` archives) and is not available through telemetry interfaces.

**Sources:**
- SCS SDK headers (scs_sdk_1_4/include/): scssdk.h, scssdk_value.h, scssdk_telemetry.h, scssdk_telemetry_channel.h, scssdk_telemetry_event.h
- ETS2-specific SDK headers: eurotrucks2/scssdk_eut2.h, eurotrucks2/scssdk_telemetry_eut2.h
- Community plugin registration code: ets2-telemetry/src/ets2-telemetry.cpp
- SCS Software official channels — announcement of telemetry SDK release
- nlhans/ets2-sdk-plugin — community plugin README and source code

### 9.2 Telemetry Data Versus Complete Navigation Route

Telemetry data can expose destination-related information such as destination city, estimated distance, and ETA. However, telemetry does not establish that it exposes the complete road-by-road route. The distinction is important: telemetry provides high-level state information (position, speed, fuel, destination city/ETA), but the detailed navigation graph — road-by-road routing data — is stored in the game's definition files (`.sii` within `.scs` archives) and is not fully available through telemetry interfaces.

**SDK-documented fields**: The SCS telemetry SDK explicitly documents player world position (x, y, z), heading/yaw, speed, fuel level, and engine RPM. Fields such as destination city name, ETA, and distance to destination are reported by community plugins (e.g., nlhans/ets2-sdk-plugin) but are not universally defined in the SCS SDK channel specification.

**Distinctions important for navigation**:
- Job destination city is not a destination waypoint coordinate.
- A route-distance value (e.g., `routeDistance`) is a scalar distance value, not a road-by-road route.
- Player position and heading are not a navigable route graph.
- Telemetry output does not establish that the application can modify the game's navigation state.

**Information not established by the SDK**: Route geometry (road-by-road routing), waypoint coordinates, AI traffic information, road classifications, or any navigable path data. The SDK documentation does not describe or support these data types. The detailed navigation graph is stored in game definition files (`.sii` within `.scs` archives) and is not available through telemetry interfaces.

**Navigation capability distinctions**:
- Reading current player position and movement: Available via telemetry (position x,y,z, heading, speed).
- Reading destination city, distance and ETA: Reported by community plugins (e.g., nlhans/ets2-sdk-plugin), but not universally defined in the SCS SDK channel specification.
- Reading destination waypoint coordinates: No dedicated destination-waypoint coordinate channel was identified in the inspected SDK version.
- Reading complete road-by-road route geometry: Not available through telemetry interfaces; requires extraction from game definition files (.sii within .scs archives).
- Modifying the game's navigation destination: No publicly available interface supports modifying ETS2's navigation destination or providing two-way navigation synchronisation.
- Two-way navigation synchronisation: No publicly available interface supports modifying ETS2's navigation destination or providing two-way navigation synchronisation. Interfaces exist for reading player state and navigation data, but not for sending destinations back to the game.

**Information not established by the SDK**: Route geometry (road-by-road routing), waypoint coordinates, AI traffic information, road classifications, or any navigable path data. The SDK documentation does not describe or support these data types. The detailed navigation graph is stored in game definition files (`.sii` within `.scs` archives) and is not available through telemetry interfaces.

**Sources:**
- SCS telemetry SDK documentation (field specification)
- Community telemetry tool reports (2024-2026)
- nlhans/ets2-sdk-plugin — community plugin shared-memory output

### 9.3 Shared Memory Interfaces

Community-developed shared memory interfaces allow external applications to read ETS2 game state. Available data reported includes: player position (x, y, z), heading, speed, fuel level, engine RPM, and some navigation information.

**Limitations:** Not officially supported. Interfaces may break between game updates. Requires reverse engineering for each game version and is not endorsed by SCS Software.

**Sources:**
- Community modding forums
- Mod developer reports

### 9.4 Telemetry Plugins

Some ETS2 mod platforms (e.g., ets2mp, multiplayer platforms) include telemetry interfaces. Data reported is similar to shared memory interfaces: position, heading, speed, fuel, engine state.

**Note:** These interfaces are not officially supported by SCS Software and may vary between platforms and game versions.

**Sources:**
- Community mod platform documentation

### 9.5 HaulInfo Telemetry Integration Goals

HaulInfo should aim to integrate with ETS2 telemetry where supported, but must clearly distinguish between:
- Telemetry-provided data (position, speed, fuel, ETA, destination city).
- Complete navigation-route geometry (road-by-road graph), which requires extraction from game definition files and is not available via telemetry.

Any companion app would need to rely on a combination of telemetry data (for current state) and locally extracted map data (for route context), understanding that the two sources serve different purposes and have different reliability characteristics.

**Sources:**
- SCS telemetry SDK documentation
- Community telemetry tool reports (2024-2026)
- Mod developer reports
## 10. Offline Extraction, Caching and Update Detection

### 10.1 Local Resource Enumeration

Resource enumeration starts from the discovered or user-selected ETS2 installation directory. Only that directory and its relevant subdirectories are accessed. No exhaustive computer-wide scanning is performed.

### 10.2 Active Map Configuration Detection

Relevant `.scs` files in the ETS2 installation directory are enumerated. Definition files are parsed using evidence-backed methods only. No assumption is made that all mods are accessible from the installation directory alone.

### 10.3 Local Extraction Approaches

**Direct access:** Read `.scs` archive headers and indexes, decompress `.sii` files on demand without writing to disk. This is the preferred approach. **Technical feasibility not yet verified: it depends on confirming the `.scs` archive structure and `.sii` format, which Sections 2.1 and 2.4 currently identify as unverified community reverse engineering. The ability to read archive indexes or decompress files directly is not yet a demonstrated HaulInfo capability.**

Extraction to a temporary disk directory is not an approved approach for HaulInfo. Any disk-based extraction would require explicit user approval and is a future design decision.

### 10.4 Incremental Versus Full Rebuilds

- **Incremental:** Update only changed data. Requires detecting changes in locally observed file modification times and definition content changes.
- **Full rebuild:** Re-extract and re-process all content. Simpler but computationally more expensive.

### 10.5 Cache Invalidation

Any cached or extracted data is held temporarily in memory during a running session and is discarded when the session ends or when no longer needed. HaulInfo does not persistently store caches, extraction results, or filesystem indexes.

Temporary in-memory data may be discarded or recalculated during a running session if:
- An `.scs` file modification time changes.
- Definition file content changes.
- A game version update is detected.

Persistent cache invalidation, profile monitoring, or cache storage between sessions is not an approved feature and requires explicit user approval before implementation.

### 10.6 Version and Mod Change Detection

**Detection methods:**
- `.scs` archive timestamp comparison (in-memory during a session only).
- Definition file content comparison (in-memory during a session only).
- Game executable version detection (in-memory during a session only).

Any comparison requiring persistent records between sessions — such as saved installation paths, filesystem inventories, or prior file modification times — is a future design decision requiring explicit user approval. HaulInfo does not persistently store such records.

### 10.7 Failure Recovery and Preservation of Existing Valid Data

Extracted `.sii` file contents and processed graph data are held temporarily in memory during a running session. They are not persistently backed up. On extraction or processing failure, HaulInfo may discard the temporary in-memory data and fall back to processing base-game-only content, or retry the operation without relying on previously cached results.

No persistent database, cache, or backup of extracted game data is created or assumed.

### 10.8 Background Processing Requirements

Background processing during a running session may include:

- Archive reading and decompression (in-memory).
- `.sii` file parsing (in-memory).
- Road graph construction (in-memory).

HaulInfo does not assume a persistent database or persistent storage architecture. Database writes or any persistent storage of extracted data require explicit user approval and are a future design decision.
## 11. Legal and Licensing Considerations

### 11.1 SCS Game Asset Extraction and Redistribution

**Technical capability versus legal permission:** Extraction of game assets is technically feasible through archive format analysis. Whether such extraction is legally permissible depends on applicable copyright law, SCS Software's terms of use, and EULA provisions. No legal conclusion is offered regarding permissibility. Redistribution of modified game assets may violate copyright.

**Sources:**
- SCS Software Terms of Use: https://www.scssoftware.com/terms
- EULA review: https://www.scssoftware.com/eula (verified November 2026)

### 11.2 Copyright and Derivative Assets

**Copyright:** ETS2 game assets (models, textures, definition files) are copyrighted by SCS Software. Derivative works created from extracted data may infringe copyright.

**Sources:**
- Copyright law
- SCS Software EULA

### 11.4 Mod Author Licences

**Individual mod authors:** Mods on Workshop and other platforms are released under various licences (MIT, GPL, commercial, "all rights reserved"). Each mod's licence must be examined individually.

**Examples:**
- rigdeck (Matozanato): MIT licence — permissive reuse with attribution. Verified against current repository.
- TruckNav-Sim: The v0.3.2 release announced a change to GPL-3.0. The current LICENSE file contains GPL-3.0.
- Truck Companion: License requires verification against current repository state.

**Sources:**
- Individual mod licence files
- Workshop page licence declarations

### 11.5 Reverse Engineering and Interoperability

**Legal status:** Reverse engineering for interoperability may be protected under fair use/digital copyright exceptions in some jurisdictions, but is not clearly established for game modding. The DMCA anti-circumvention provisions (US) may apply.

**Sources:**
- EFF fair use analysis
- Copyright law experts
- Jurisdiction-specific legal advice recommended

### 11.6 Licences of Existing Open-Source Projects

- **rigdeck (Matozanato):** MIT licence — permissive reuse with attribution. Verified against current repository.
- **TruckNav-Sim (Rares-Muntean):** The v0.3.2 release announced a change to GPL-3.0. The current LICENSE file contains GPL-3.0.
- **Truck Companion (Nethercap):** License requires verification against current repository state.

**Sources:**
- Repository license files

### 11.7 Unresolved Legal Questions

- Whether extraction and analysis of game assets for personal use constitutes fair use.
- Whether derivative map data from extracted game files is a copyright infringement.
- Whether Steam Workshop content licences allow incorporation into other software.
- Jurisdiction-specific applicability of anti-circumvention laws.

**Recommendation:** Consult legal counsel specializing in software copyright and game modding before making redistribution claims.

**Sources:**
- EFF fair use analysis
- Copyright law experts
- Jurisdiction-specific legal advice
## 12. Performance Considerations

No unsupported numerical estimates (e.g., "20,000–30,000 base road segments", "100,000+ ProMods segments", "500,000+ combined segments", "200,000+ nodes") will be included. Specific database sizes or extraction times will not be claimed unless a reproducible measurement, exact test environment, methodology, and source are provided. If the values cannot be verified, they are removed. Performance factors are explained qualitatively instead.

### 12.1 Archive Reading and Decompression

**Performance:** LZMA compression offers better compression but slower decompression; LZ4 is faster but less compact. Reading individual files from `.scs` archives without full decompression is preferred for incremental processing.

**Sources:**
- scs_t performance data (2024-2026, unverified — scs_t archive tool repository and licence not yet identified)
- Community benchmarking

### 12.2 Road Graph Construction

**Complexity:** O(n) where n is the number of road segments/nodes. Graph construction involves parsing `.sii` files, building node-edge relationships, and validating topology.

**Optimization:** Cache parsed data, skip unchanged sections, use efficient data structures (adjacency lists, index structures).

**Sources:**
- Community graph construction benchmarks (2024-2026)
- Modding tool performance data

### 12.3 Database Size and Access

**Expected database sizes:** Specific sizes cannot be verified without a reproducible measurement, exact test environment, methodology, and source. If verified measurements become available, they should be documented with full methodology. Performance characteristics should be described qualitatively: read-heavy access patterns (route calculation), write-heavy initial extraction, and cache update operations.

**Sources:**
- Community project reports (2024-2026) — only as unverified references, not claims

### 12.4 Modest Windows Hardware

**Target configuration:** Typical Windows 10/11 home system with integrated or modest discrete graphics, 8GB RAM, modern CPU.

**Performance constraints:** Must complete extraction/cache generation in reasonable time. Must not cause noticeable ETS2 performance degradation if running concurrently.

**Sources:**
- Target user hardware assumptions
- Community performance data (2024-2026)
## 13. Phone Companion and Future Integration Considerations

### 13.1 ETS2 Information Useful to a Companion App

**Player state:** Position (x, y, z), heading, speed, fuel level, engine RPM.

**Navigation state:** Currently unavailable via supported interfaces (see Section 9). Any companion app would need to rely on unofficial reverse-engineered interfaces, which are game-version-specific and unstable.

**Fuel and engine state:** Reported by some community telemetry tools, but not officially documented.

**Sources:**
- Community telemetry tool reports (2024-2026)
- Mod developer reports

### 13.2 What Should NOT Be Designed in Phase 2

- Mobile application frameworks.
- PC-to-phone communication protocols (beyond basic design planning).
- Network security architecture.
- Mobile UI design.
- Phone application implementation.

These belong to later phases.

**Sources:**
- This task specification Section 10
## 14. Technical Conclusions

**Confirmed findings:**
1. ETS2 installations can be discovered through known installation locations and local metadata, without Steam account access or online Steam APIs.
2. `.scs` archives may contain `.sii` definition files and binary asset data. (Based on community reverse engineering of SCS archive format; not verified against primary SCS documentation.)
3. Road network data is stored in text-based SCS definitions with road segments, nodes, and connections. (Based on community reverse engineering of `.sii` definition files; not verified against primary SCS documentation.)
4. DLCs may add additional `.scs` archive files within the ETS2 installation directory. Assignment of specific archive files to specific DLCs is not established without a verified identification method.
5. No destination-modification interface was identified in the inspected public SDK and community-plugin sources. Interfaces exist for reading player state and navigation data, but not for sending destinations back to the game.
6. Existing open-source projects (rigdeck, TruckNav-Sim) provide partial extraction capabilities under verified licences.

**Likely findings (unverified but probable):**
1. `.sii` files use text-based format (SCS-specific text format; not JSON).
2. Road segments reference nodes by ID, forming a graph structure.
3. Roundabouts and interchanges have special encoding in node definitions.
4. Coordinate transformation to external frameworks requires undocumented scaling factors.
5. The ETS2 coordinate system uses metres as the likely unit (based on observed coordinate ranges), but the exact origin, orientation, and scale remain unverified.

**Unknown findings:**
1. Exact `.sii` file format specification (field names, data types, structure).
2. Precise coordinate origin, orientation, and scale.
3. Complete road network coverage within `.scs` archives (base game vs. DLC vs. mods).
4. How DLC combinations interact at the data level.
5. 3D model format details and level-of-detail implementation.
6. Elevation data sources and precision.
7. Any supported interface for reading/modifying ETS2 navigation state.

**Sources:**
- Community reverse engineering (2024-2026)
- SCS Software forum announcements (where available)
- Modding tool development activity
- Open-source project source code inspection
## 15. Unresolved Questions and Research Gaps

1. What is the exact format and organisational structure of `.sii` definition files?
2. How complete is the road network data within `.scs` archives (base game vs. DLC vs. mods)?
3. What coordinate transformation is needed for map integration, and what is the scaling factor?
4. How do DLC combinations affect map area and data overlap?
5. What 3D model formats are used, and can they be read without the ETS2 engine?
6. What elevation data is available, and how precise is it?
7. Can HaulInfo reliably discover ETS2 installations without user intervention (all Steam configurations)?
8. What legal clearance is needed for incorporating extracted game data into HaulInfo?
9. What performance characteristics should HaulInfo target for modest Windows hardware?
10. Are there any supported interfaces for reading ETS2 navigation state (confirmed or denied)?

**Sources:**
- Ongoing community analysis (2024-2026)
- SCS Software forum announcements
- Modding tool development activity
- Community reverse engineering projects
## 16. Sources

- SCS Software modding wiki: https://modding.scssoft.com/wiki/Main_Page
  - Supports: General ETS2 modding documentation, archive format overview, telemetry SDK references
- SCS Software Terms of Use: https://www.scssoftware.com/terms
  - Supports: Legal considerations for game asset extraction (Section 11.1)
- SCS Software EULA: https://www.scssoftware.com/eula
  - Supports: Legal considerations for game asset extraction (Section 11.1)
- rigdeck (Matozanato): https://github.com/Matozanato/rigdeck
  - Supports: Open-source ETS2 road network extraction tool (Sections 8.1, 11.6, 14)
- TruckNav-Sim (Rares-Muntean): https://github.com/Rares-Muntean/TruckNav-Sim
  - Supports: ETS2 navigation tool and licensing information (Sections 8.2, 11.6, 14)
- Truck Companion (Nethercap): https://github.com/Nethercap/truck-companion
  - Supports: ETS2 companion app reference (Sections 8.3, 11.6) — licence requires current verification
- SCS SDK plugin (nlhans): https://github.com/nlhans/scs-sdk-plugin
  - Supports: SCS telemetry SDK and community plugin implementation (Section 9)
- RenCloud scs-sdk-plugin: https://github.com/RenCloud/scs-sdk-plugin
  - Supports: SCS telemetry SDK reference (Section 9)

The following claims in this document rely on vague or non-traceable community sources ("Community analysis...", "Modding documentation", "Community reverse engineering...") without identifiable URLs or named publications. These claims are therefore treated as unverified:

- Exact `.scs` archive internal structure (Section 2.1)
- Exact `.sii` file structure and content (Sections 2.4, 3-5)
- Road network definitions and connectivity details (Sections 3-5)
- Map sector organisation (Section 4.2)
- Coordinate units, transformations, and origin (Sections 3.1, 3.2, 5.1, 5.2)
- DLC archive-to-DLC assignment (Sections 7.2, 14)
- Mod conflict resolution behaviour (Section 7.3)

No Steamworks or Steam Workshop integration research is included in this source list, as HaulInfo does not integrate with Steam Workshop.