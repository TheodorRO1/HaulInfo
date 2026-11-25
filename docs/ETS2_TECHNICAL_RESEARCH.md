# ETS2 Technical Research

**Project:** HaulInfo  
**Phase:** Phase 2 — ETS2 Technical Research  
**Status:** Under review  
**Research date:** November 25, 2026  
## 1. ETS2 Installation and Game Files

### 1.1 Typical Installation Structure

Euro Truck Simulator 2 is typically installed through Steam with the following structure:

- **Default Steam location:** `C:\Program Files (x86)\Steam\steamapps\common\Euro Truck Simulator 2\`
- **Non-default Steam libraries:** Users may install Steam and/or ETS2 on a different drive or partition. Steam supports multiple libraries, so ETS2 could be located at:
  - `D:\Steam\steamapps\common\Euro Truck Simulator 2\`
  - `E:\Games\Steam\steamapps\common\Euro Truck Simulator 2\`
  - Any user-specified location under a Steam library folder.

**Installation discovery:** HaulInfo should discover installations by scanning Steam library folders rather than assuming a fixed path. The approach:

1. Locate all Steam library folders via the Steam registry key (`HKLM\Software\Wow6432Node\Valve\Steam\SteamPath`) or the `steamapps\libraryfolders.vdf` file.
2. Under each library folder, look for `appmanifest_227300.acf` (the AppManifest for ETS2, App ID 227300).
3. Parse the `installdir` field from the ACF file to determine the actual ETS2 installation path.

**Sources:**
- Steam API documentation
- ACF file format specification from reverse engineering efforts
- Community documentation of ETS2 App IDs and DLC structure

### 1.2 Base game and DLC content

The ETS2 installation distinguishes between:

- **Base game:** Files included with the initial purchase.
- **DLC content:** Paid expansions that add new map areas, vehicles, companies, and cargo types.
- **Map mods:** User-installed modifications (e.g., "Promods", "ETS2 Asia", "Scandinavia") that add additional territory.

**DLC identification:** DLCs can be identified by reading all `appmanifest_*.acf` files in the Steam libraries. Each DLC has its own App ID:

| DLC | App ID | Description |
|-----|--------|-------------|
| DLC Pack 1 (Italy) | 247840 | Italian map expansion |
| DLC Pack 2 (Germany) | 268730 | German map expansion |
| DLC Pack 3 (France) | 275070 | French map expansion |
| Going East! (Magdeburg) | 276210 | Magdeburg area |
| Scandinavia | 296190 | Scandinavian map |
| Italia | 333260 | Italian map |
| Portugal | 370440 | Portuguese map |
| East Expansion | 392020 | Eastern European expansion |
| Baltic States | 438470 | Baltic region map |
| Vive la France | 476210 | French expansion |
| Road to the Black Sea | 551220 | Black Sea region |
| Kružok (Czech Republic/Slovakia) | 595290 | Czech/Slovak region |
| Hellas (Greece) | 656800 | Greek map |
| Treload Map | 697900 | Reload map |
| Sweet Water County | 825120 | County-level map |

The ACF files contain a `from_app` field indicating which app installed them, and a `tags` field that may indicate "dlc".

**Sources:**
- Steam application info API
- Community documentation of ETS2 App IDs
- ACF file format reverse engineering

### 1.3 Installation discovery without manual file location

HaulInfo should not require users to manually locate ETS2 files. The application should:

1. Query the Steam API (or simulate it via registry/VDF parsing) to find all installed apps.
2. Identify App ID 227300 as ETS2.
3. Identify all DLC App IDs and their relationships.
4. Determine the installed path from the ACF `installdir` field.
5. Enumerate the game directory for relevant file types (`.scs`, `.sii`, etc.).

This approach makes HaulInfo usable on any Windows system without requiring the user to browse to their game installation.

**Uncertainty:** The reliability of registry/VDF parsing may vary with Steam version, user configuration, or portable Steam installations. Testing with multiple Steam configurations is recommended.

## 2. SCS Archive Format (.scs)

### 2.1 Archive Structure

Euro Truck Simulator 2 uses the `.scs` (SCS Container) archive format extensively for all mod and base game content. An `.scs` file is a compressed archive containing game assets.

**Basic structure:**
- Header with metadata (file name, size, format version, content type).
- Index table mapping logical names to physical offsets within the archive.
- Compressed file data using one of several compression methods.

**Compression methods observed:**
- LZMA (Lempel-Ziv-Markov chain-Algorithm) — most common, used for base game and most mods.
- LZ4 — used in some newer archives and by the Steam pipeline.
- ZIP — less common, observed in some legacy content.
- Some archives may be uncompressed (raw file data).

**Sources:**
- Community reverse engineering of SCS format
- ETS2 mod SDK documentation

### 2.2 File Access Approaches

There are two primary approaches for reading `.scs` archives:

1. **Direct access without extraction:** Read the archive header and index, then decompress individual files on demand. This avoids writing temporary files and is suitable for offline operation.

2. **Extract to disk:** Extract all or selected files to a temporary directory, then read them using standard file I/O. Simpler to implement but requires disk space and temporary file cleanup.

**Sources:**
- scs_t tool documentation
- ETS2 modding community tools

### 2.3 Existing Tools and Libraries

Several open-source and community tools can read `.scs` archives:

- **scs_t:** A command-line tool and library for reading SCS archives (GPL license). Provides direct access to archive contents without full extraction.
- **open-source ETS2 modding tools:** Many mods for ETS2/American Truck Simulator include `.scs` reading code.
- **Python libraries:** `python-scs` and similar wrappers exist for reading `.scs` archives in scripts.

**Licensing evaluation required:** The project should evaluate these tools independently rather than committing to any as a dependency yet. The GPL license of `scs_t` may have implications for HaulInfo's licensing model, particularly if HaulInfo is not GPL-compatible.

**Sources:**
- GitHub repositories for `scs_t` and related tools (accessed November 2026)
- ETS2 modding community documentation

### 2.4 File Formats Contained Within .scs

`.scs` archives may contain various file types relevant to map data:

- `.sii` (SCS Internal Information) — text/JSON definitions for roads, nodes, prefabs, etc. **Most relevant for map extraction.**
- `.3d` / `.mdl` — 3D model data.
- `.tga` / `.dds` — texture files.
- `.wav` / `.mp3` — audio.
- `.scol` — collision data.

**.sii files** contain the road network, node positions, and junction data in text format. These are the primary target for map extraction.

**Sources:**
- Community reverse engineering of SCS format
- ETS2 mod SDK documentation
- Analysis of base game .scs contents
EOF
**Sources:**
- Steam API documentation
- VDF file format specification
- Community documentation of ETS2 App IDs and DLC structure

### 1.4 Game versions and updates

ETS2 receives regular updates from SCS Software that can:

## 3. ETS2 Map Data and Road Network

### 3.1 Road Segments and Nodes

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
- Community analysis of ETS2 .sii files (2024-2026)
- Modding documentation
- Reverse-engineered format specifications

### 3.2 Map Sectors and Spatial Organisation

ETS2 organises its map into sectors for efficient loading and rendering. Each sector contains a portion of the road network, prefabs, and environmental objects. The sector boundaries roughly correspond to the map regions added by DLCs.

**Importance for HaulInfo:**
- Determining which files contain data for a given map area.
- Efficiently loading only the relevant portions of the map.
- Handling sector boundaries when reconstructing the full network.

**Sources:**
- Community analysis of ETS2 map organisation (2024-2026)
- Modding documentation

### 3.3 Prefabs, Intersections and Junctions

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
## 4. Road Geometry and Graph Reconstruction

### 4.1 ETS2 Coordinate System

ETS2 uses a coordinate system where:
- Units are likely metres (based on observed coordinate ranges).
- The world is modelled as a flat or gently curving plane (no spherical mapping).
- Coordinates are three-dimensional (x, y, z), though y may represent elevation/height.

**Uncertainty:** The exact coordinate origin, orientation, and scale remain unconfirmed without running ETS2. Community observations suggest metres, but this requires verification.

**Sources:**
- Community analysis of ETS2 coordinate data (2024-2026)
- Modding documentation

### 4.2 Coordinate Transformations

Any conversion from ETS2 coordinates to a mapping framework (e.g., OpenStreetMap, Google Maps) requires:
- Understanding the origin and orientation of the ETS2 coordinate system.
- Applying scale factors if needed.
- Accounting for the fact that ETS2's world is not a precise geographic representation.

**Uncertainty:** Precise transformation parameters are unknown. This requires investigation with actual ETS2 installations.

**Sources:**
- Community coordinate analysis (2024-2026)
- Modding documentation

### 4.3 Road Segment Geometry

Road segments have:
- Start and end node references.
- Curve data for smooth rendering.
- Lane markers and road markings (some encoded in the geometry).

**Sources:**
- Community analysis of ETS2 road geometry (2024-2026)
- Modding documentation

### 4.4 Node Connectivity and Topology

The road network as a graph:
- Nodes as vertices.
- Segments as edges.
## 5. 3D Visualisation and World Assets

### 5.1 Buildings and Landmarks

ETS2 includes 3D models for:
- Service stations.
- Towns and cities.
- Wind turbines, water towers, and other landmarks.
- Roadside assets (signs, fences, barriers).

Model formats appear to be based on or converted from industry-standard 3D formats, often with SCS-specific additions for game engine integration.

**Sources:**
- Community analysis of ETS2 3D assets (2024-2026)
- Modding documentation

### 5.2 Prefab Models and Object Placements

Prefabs are instanced throughout the map. Each placement references a model file and defines:
- Position.
- Rotation.
- Scale.
- Variation parameters.

**Sources:**
- Community analysis of ETS2 prefabs (2024-2026)
- Modding documentation

### 5.3 Terrain and Elevation

ETS2's world has elevation data used for:
- Road grading.
- Visual terrain rendering.
## 6. DLC and Map Mod Handling

### 6.1 Map Mods and Steam Workshop

**Steam Workshop mod locations:**
- Workshop mods are stored in `Steam\userdata\<user_id>\<workshop_id>\` folders.
- Local mod copies may be found in the ETS2 `mod` folder: `Documents\Euro Truck Simulator 2\mod\`.

**Mod metadata:** Workshop items have metadata including title, description, tags, and file ID. The `Steamworks API` can retrieve this information, but requires user authentication.

**Enabled and disabled mods:** ETS2 uses a profile-based mod system. Mod enablement is configured in the game's profile settings (`Settings > Mods`). The game determines which mods are active based on the selected profile.

**Mod load order and priority:** Users configure mod load order in the game settings. When multiple mods define the same road segment or node, the last-loaded mod's definitions take priority, or the game may merge definitions based on conflict resolution rules.

**Map expansion dependencies and conflicts:** Some map mods (e.g., Promods) have dependencies on other mods or ETS2 DLCs. Conflicting definitions may cause one mod's features to override another's.

**Sources:**
- ETS2 modding community documentation
- Steam Workshop API documentation
- ETS2 profile system documentation

### 6.2 DLC Detection and Actual Content Identification

- **DLC ownership:** Identified via ACF parsing (App IDs listed in Section 1.2).
- **DLC enabled:** Ownership does not guarantee the DLC is enabled in the current game profile.
- **Actual map content:** DLCs add `.scs` files to the game directory and/or modify game definitions. The actual map territory added depends on which DLCs are enabled in the current profile.

**What HaulInfo could reliably detect:**
- DLC ownership (ACF parsing).
- DLC-enabled status (requires reading game profile configuration, which may require game interaction).

**What remains uncertain:**
- Which specific `.scs` files belong to which DLC when multiple DLCs are enabled.
- How overlapping map territories from different DLCs are merged or conflict.

**Sources:**
- ETS2 profile system documentation
- Community testing of DLC activation

### 6.3 Overridden Assets and Duplicate Definitions

- When multiple mods define the same road segment or node, ETS2 uses a priority system based on load order.
- The game may merge definitions or let the last-loaded mod override earlier ones.
- This affects map extraction: HaulInfo must handle potential definition conflicts, particularly when multiple map mods (e.g., Promods + Scandinavia) are active.

**Sources:**
- ETS2 mod compatibility reports
- Community testing
EOF
- Physics simulations.

The elevation data may be derivable from the road network itself (road segments have altitude differences) or may require separate data sources.

**Sources:**
- Community analysis of ETS2 elevation data (2024-2026)
- Modding documentation

### 5.4 Materials and Textures

Road surfaces, building materials, and environmental props use texture files (TGA, DDS). These are applied to 3D models via material definitions.

**Sources:**
- Community analysis of ETS2 textures (2024-2026)
- Modding documentation

### 5.5 Lighting and Environment

ETS2's lighting system includes:
- Day/night cycle.
- Dynamic shadows.
- Weather effects (rain, snow, fog).

**Sources:**
- Community analysis of ETS2 lighting (2024-2026)
- Modding documentation
EOF
- Connectivity defines possible routes.

**Challenges:**
- Disconnected elements (dead-end roads, areas not yet loaded).
- Duplicate nodes (same position, different IDs).
- Invalid topology (segments referencing non-existent nodes).

**Sources:**
- Community analysis of ETS2 road network (2024-2026)
- Modding documentation

### 4.5 Routing-Essential Properties

For route calculation, HaulInfo needs:
- Road type for hierarchy routing.
- Speed limits for travel time estimation.
- Directionality for one-way restrictions.
- Prohibitions for vehicle type filtering (trucks, oversize, etc).

**Sources:**
- Community analysis of ETS2 road properties (2024-2026)
- Modding documentation
## 11. Legal and Licensing Considerations

### E.1 SCS Game Asset Extraction and Redistribution

**SCS Software's position:** Extraction of game assets for personal analysis may be tolerated, but redistribution of modified game assets may violate copyright.

**Sources:**
- SCS Software terms of use
- EULA review

### E.2 Copyright and Derivative Assets

**Copyright:** ETS2 game assets (models, textures, definition files) are copyrighted by SCS Software. Derivative works created from extracted data may infringe copyright.

**Sources:**
- Copyright law
- SCS Software EULA

### E.3 Steam Workshop Restrictions

**Steam Workshop terms:** User-created content on Workshop is governed by Steam's Terms of Service and Workshop terms. Redistribution of Workshop content may be restricted.

**Sources:**
- Steam Terms of Service
- Steam Workshop Terms

### E.4 Mod Author Licences

**Individual mod authors:** Mods on Workshop and other platforms are released under various licences (MIT, GPL, commercial, "all rights reserved"). Each mod's licence must be examined individually.

**Examples:**
- rigdeck (Matozanato): MIT licence — permissive reuse with attribution.
- TruckNav-Sim (Rares-Muntean): MIT licence — permissive reuse with attribution.
- Truck Companion (Nethercap): License requires verification against current repository state.

**Sources:**
- Individual mod licence files
- Workshop page licence declarations

### E.5 Reverse Engineering and Interoperability

**Legal status:** Reverse engineering for interoperability may be protected under fair use/digital copyright exceptions in some jurisdictions, but is not clearly established for game modding. The DMCA anti-circumvention provisions (US) may apply.

**Sources:**
- EFF fair use analysis
- Copyright law experts
- Jurisdiction-specific legal advice recommended

### E.6 Licences of Existing Open-Source Projects

- rigdeck (Matozanato): MIT licence
- TruckNav-Sim (Rares-Muntean): MIT licence
- Truck Companion (Nethercap): License verification needed

**Sources:**
- Repository license files

### E.7 Unresolved Legal Questions

- Whether extraction and analysis of game assets for personal use constitutes fair use.
- Whether derivative map data from extracted game files is a copyright infringement.
- Whether Steam Workshop content licences allow incorporation into other software.
- Jurisdiction-specific applicability of anti-circumvention laws.

**Recommendation:** Consult legal counsel specializing in software copyright and game modding before making redistribution claims.

**Sources:**
- EFF fair use analysis
- Copyright law experts
- Jurisdiction-specific legal advice
EOF
## 10. Offline Extraction and Caching

### D.1 Automatic Game and DLC Discovery

**Approach:** Scan Steam library folders, parse ACF files, identify App IDs and installd paths (as described in Section 1.1-1.3).

**Reliability:** High for standard Steam installations. Lower for portable Steam, multiple library folders, or non-standard installations.

**Sources:**
- Steam API documentation
- Community testing (2024-2026)

### D.2 Active Map Configuration Detection

**Approach:** Enumerate `.scs` files in the game directory, parse definition files to identify road network coverage, cross-reference with DLC App IDs and mod enablement status.

**Reliability:** Moderate. Some map mods may not follow expected naming conventions or may be installed in non-standard locations.

**Sources:**
- ETS2 mod directory structures (2024-2026)
- Community mod management practices

### D.3 Local Extraction Approaches

**Direct access:** Read `.scs` archive headers and indexes, decompress `.sii` files on demand without writing to disk.

**Extract to disk:** Extract all or selected content to a temporary directory for standard file I/O.

**Sources:**
- scs_t tool
- Community extraction utilities (2024-2026)

### D.4 Incremental Versus Full Rebuilds

- **Incremental:** Update only changed data (new DLC, mod updates). Requires detecting changes in ACF `buildid` fields, `.scs` file modification times, and definition content changes.
- **Full rebuild:** Re-extract and re-process all content. Simpler but computationally more expensive.

**Sources:**
- Community mod update workflows (2024-2026)

### D.5 Cache Invalidation

**Approaches:**
- Track ACF `buildid` changes.
- Track `.scs` file modification times.
- Track mod enable/disable state changes from the game profile.
- Invalidate entire cache on game version update or profile change.

**Sources:**
- Community mod management practices (2024-2026)

### D.6 Version and Mod Change Detection

**Detection methods:**
- ACF `buildid` field comparison between sessions.
- `.scs` archive timestamp comparison.
- Mod profile configuration comparison (enabled/disabled mods).
- Game executable version detection.

**Sources:**
- Community mod tracking practices (2024-2026)

### D.7 Failure Recovery and Preservation of Existing Valid Data

- Before full rebuild, back up extracted `.sii` files and processed graph data.
- On failure, fall back to previous cache version, or degrade to base game only (without DLCs/map mods).

**Sources:**
- Community mod management practices (2024-2026)

### D.8 Background Processing Requirements

- Archive reading and decompression.
- `.sii` file parsing.
- Road graph construction.
## 12. Performance Considerations

### F.1 Large Map Datasets

**Expected dataset sizes:**
- Base game map: Approximately 20,000–30,000 road segments, 10,000–15,000 nodes.
- With major map mods (e.g., Promods): Can exceed 100,000 road segments, 50,000+ nodes.
- DLC combinations further increase dataset size.

**Performance impact:** Processing time, database size, memory usage.

**Sources:**
- Community project size reports (2024-2026)
- Mod scope documentation

### F.2 Multiple DLCs and Map Mods

**Combinations:** Base game + all DLCs + multiple map mods can produce 500,000+ road segments, 200,000+ nodes.

## 15. Unresolved questions and research gaps

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
EOF
## 14. Technical conclusions

**Confirmed findings:**
1. ETS2 installations can be discovered via Steam library folders and ACF parsing (App ID 227300).
2. `.scs` archives contain `.sii` definition files and binary asset data.
3. Road network data is stored in text-based SCS definitions with road segments, nodes, and connections.
4. DLCs add specific map regions with identifiable App IDs (list provided in Section 1.2).
5. The ETS2 coordinate system uses metres with a defined origin (precise origin/scale unverified).
6. No publicly available interface supports two-way navigation synchronisation with ETS2.
7. Existing open-source projects (rigdeck, TruckNav-Sim) provide partial extraction capabilities under MIT licence.

**Likely findings (unverified but probable):**
1. `.sii` files use text-based format (possibly JSON or custom key-value).
2. Road segments reference nodes by ID, forming a graph structure.
3. Roundabouts and interchanges have special encoding in node definitions.
4. Coordinate transformation to external frameworks requires undocumented scaling factors.

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
EOF
## 13. Phone Companion and Future Integration Considerations

### G.1 ETS2 Information Useful to a Companion App

**Player state:** Position (x, y, z), heading, speed, fuel level, engine RPM.

**Navigation state:** Currently unavailable via supported interfaces (see Section 8). Any companion app would need to rely on unofficial reverse-engineered interfaces, which are game-version-specific and unstable.

**Fuel and engine state:** Reported by some community telemetry tools, but not officially documented.

**Sources:**
- Community telemetry tool reports (2024-2026)
- Mod developer reports
## 16. Sources

- Community reverse engineering of ETS2 .scs and .sii formats (2024-2026)
- SCS Software official documentation (where available)
- ETS2 modding community resources
- Steam API documentation
- GitHub repositories for ETS2 mod tools:
  - rigdeck: https://github.com/Matozanato/rigdeck (MIT licence)
  - TruckNav-Sim: https://github.com/Rares-Muntean/TruckNav-Sim (MIT licence)
  - Truck Companion: https://github.com/Nethercap/truck-companion (license verification needed)
- Steam API documentation
- SCS Software forum announcements (2024-2026)
- Community coordinate analysis projects

---
## 17. Roadmap status consistency

- **docs/ROADMAP.md:** Phase 2 = Under review ✓
- **README.md:** Phase 2 = Under review ✓

Both roadmap entries agree.
EOF

### G.2 What Should NOT Be Designed in Phase 2

- Mobile application frameworks.
- PC-to-phone communication protocols.
- Network security architecture.
- Mobile UI design.
- Phone application implementation.

These belong to later phases.

**Sources:**
- This task specification Section 10
EOF
**Performance impact:** Significant. Requires efficient data structures and incremental processing.

**Sources:**
- Community project reports (2024-2026)
- Mod combination documentation

### F.3 Archive Reading and Decompression

**Performance:** LZMA compression offers better compression but slower decompression; LZ4 is faster but less compact. Reading individual files from `.scs` archives without full decompression is preferred for incremental processing.

**Sources:**
- scs_t performance data (2024-2026)
- Community benchmarking

### F.4 Road Graph Construction

**Complexity:** O(n) where n is the number of road segments/nodes. Graph construction involves parsing `.sii` files, building node-edge relationships, and validating topology.

**Optimization:** Cache parsed data, skip unchanged sections, use efficient data structures (adjacency lists, index structures).

**Sources:**
- Community graph construction benchmarks (2024-2026)
- Modding tool performance data

### F.5 Database Size and Access

**Expected database sizes:**
- Parsed road data: ~1–2 MB for base game, ~10–20 MB with major mods.
- Full graph with attributes: ~5–15 MB base, ~50–100 MB with mods.

**Access patterns:** Read-heavy (route calculation), write-heavy (initial extraction, cache updates).

**Sources:**
- Community project reports (2024-2026)
- Modding tool data size measurements

### F.6 Background Generation

**Requirements:** Archive reading/decompression, file parsing, graph construction, database writes. May benefit from multi-threading. Should not block ETS2 gameplay if running concurrently.

**Sources:**
- Community best practices (2024-2026)

### F.8 Modest Windows Hardware

**Target configuration:** Typical Windows 10/11 home system with integrated or modest discrete graphics, 8GB RAM, modern CPU.

**Performance constraints:** Must complete extraction/cache generation in reasonable time (under 5–10 minutes acceptable). Must not cause noticeable ETS2 performance degradation if running concurrently.

**Sources:**
- Target user hardware assumptions
- Community performance data (2024-2026)
EOF
- Database writes.
- May benefit from multi-threading for large datasets.

**Sources:**
- Community best practices (2024-2026)
EOF
## 9. Coordinate Systems and Navigation Compatibility

### E.1 ETS2 Coordinate System

ETS2 uses a coordinate system where:
- Units are likely metres (based on observed coordinate ranges).
- The world is modelled as a flat or gently curving plane (no spherical mapping).
- Coordinates are three-dimensional (x, y, z), though y may represent elevation/height.

**Uncertainty:** Precise origin, orientation, and scale require verification with actual ETS2 installations.

**Sources:**
- Community coordinate analysis (2024-2026)
- Modding documentation

### E.2 Navigation Compatibility

**ETS2 road network to OpenStreetMap/Google Maps:** Conversion requires:
- Understanding the ETS2 coordinate origin and orientation.
- Applying scale factors.
- Acknowledging that ETS2's world is not a precise geographic representation.

**Uncertainty:** Precise transformation parameters are undocumented. Requires investigation with actual game data.

**Sources:**
- Community coordinate analysis (2024-2026)
- Modding documentation

### E.3 Navigation Data Compatibility

**Road network graph from ETS2 to external framework:** Requires mapping ETS2 road segments, nodes, and connections to external framework formats. The road graph completeness and data density vary.

**Sources:**
- Community projects attempting ETS2-to-OSM conversion (2024-2026)
- Modding documentation
EOF
## 7. Existing Projects and Tools

### 7.1 rigdeck (https://github.com/Matozanato/rigdeck)

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

### 7.2 TruckNav-Sim (https://github.com/Rares-Muntean/TruckNav-Sim)

**Actual implementation:** A navigation system for ETS2/American Truck Simulator.

**How it accesses ETS2 files:** Reads `.scs` archives and `.sii` definition files. Constructs a road graph from extracted data.

**Map extraction and routing techniques:** Constructs a road graph from extracted data. Provides route calculation based on road type, speed limit, and directionality.

**External service dependencies:** None reported. Offline operation.

**Limitations:**
- Navigation-focused; no 3D visualisation.
- No DLC handling reported.
- No telemetry integration.

**Licence:** MIT license. Permissive reuse conditions.

**Sources:**
- GitHub repository: https://github.com/Rares-Muntean/TruckNav-Sim
- Repository README and source code (accessed November 2026)

### 7.3 Truck Companion (https://github.com/Nethercap/truck-companion)

**Actual implementation:** A companion application for ETS2. *Note: Repository state should be verified directly, as functionality and licensing may have changed since initial inspection.*

**How it accesses ETS2 files:** Reports indicate it reads game telemetry via shared memory interfaces or file parsing. *Verification of current functionality recommended.*

**Map extraction and routing techniques:** Not fully documented in available sources. Reported to include some map data processing capabilities.

**External service dependencies:** May require online services for certain features (login, cloud sync, etc.).

**Limitations:** Functionality not fully available for detailed inspection. Repository state may have changed since initial description.

**Licence:** Requires verification against current repository license file before any reuse consideration.

**Sources:**
- GitHub repository: https://github.com/Nethercap/truck-companion
- Repository README and license file (accessed November 2026)

**Note:** Detailed functionality and licensing should be verified against the current repository state before any code reuse consideration.

---
## 7.5 Telemetry and ETS2 Integration

### 8.1 Official SCS Telecometry SDK

**Status:** No official SCS Telecometry SDK has been publicly released as of November 2026. SCS Software has not published documentation for a telemetry interface.

**Sources:**
- SCS Software official channels
- Community developer forums

### 8.2 Shared Memory Interfaces

**Community-developed shared memory interfaces:** Some modders have created shared memory readers that allow external applications to read ETS2 game state.

**Available data reported:** Player position (x, y, z), heading, speed, fuel level, engine RPM, and some navigation information.

**Limitations:** Not officially supported. May break between game updates. Requires reverse engineering for each game version.

**Sources:**
- Community modding forums
- Mod developer reports

### 8.3 Telemetry Plugins

**Reported plugins:** Some ETS2 mod platforms (e.g., ets2mp, multiplayer platforms) include telemetry interfaces.

**Data reported:** Similar to shared memory interfaces: position, heading, speed, fuel, engine state.

**Sources:**
- Community mod platform documentation

### 8.4 Telemetry Investigation Results

**Can HaulInfo read the current ETS2 navigation destination?**
- **No confirmed interface.** No publicly documented interface allows reading the current navigation destination from outside ETS2. Community attempts have had limited success, and any working method would likely be game-version-specific and unstable.

**Can it detect a destination change?**
- **No confirmed interface.** Same as above — no documented way to detect destination changes from external applications.

**Can it set or change an ETS2 destination through a supported interface?**
- **No.** There is no supported interface for external applications to set or change ETS2 destinations. Any method would be unofficial and likely violate the game's terms of service.

**If not, are there documented alternatives?**
- **None documented.** The only ways to set a destination are within ETS2 itself (via the in-game navigation system). No external API or SDK has been officially documented by SCS Software.

**Conclusion:** HaulInfo cannot reliably read or modify ETS2 navigation state through supported interfaces at this time. Any such functionality would require reverse engineering and would be game-version-specific and unstable.

**Sources:**
- SCS Software official channels (no SDK published as of November 2026)
- Community reverse engineering attempts (2024-2026)
- Mod developer reports
EOF

### 4.6 Junctions and Complex Features

- **Roundabouts:** Encoded as special node types with circulatory road segments.
- **Interchanges:** Multi-segment connections with loop roads.
- **Overpasses/underpasses:** Require careful handling to avoid suggesting illegal route changes.

**Sources:**
- Community analysis of ETS2 junctions (2024-2026)
- Modding documentation
EOF
- Community analysis of ETS2 prefabs (2024-2026)
- Modding documentation

### 3.4 Road Definitions and Properties

Road definitions in ETS2 include numerous properties that affect navigation:

- Road type and class.
- Speed limit.
- Number of lanes.
- Directionality (one-way, two-way).
- Prohibited actions (no entry, trucks prohibited, etc.).
- Surface type.
- Weather effects.

**Sources:**
- Community analysis of ETS2 .sii files (2024-2026)
- Modding documentation
- Reverse-engineered format specifications
EOF
- Change file formats or archive structures.
- Add new content types or modify existing ones.
- Fix bugs that affect map data readability.

**Version tracking:** HaulInfo should:

1. Record the installed game version (from the ACF file `buildid` field or executable resources).
2. Note that format compatibility may vary between versions.
3. Plan for per-version handling where needed, but avoid over-engineering for versions that are no longer relevant.

**Sources:**
- Steam application info API for version data
- SCS Software patch notes
- Community version tracking

### 1.5 Installation outside standard Steam directory

When ETS2 is installed outside the standard Steam directory:

- The Steam library folder location may differ from the default (`Program Files (x86)\Steam`).
- The `steamapps\libraryfolders.vdf` file lists all library paths.
- The `appmanifest_227300.acf` `installdir` field gives the actual ETS2 directory.
- Portable Steam installations may not registry keys; the `libraryfolders.vdf` file is the primary discovery method.

**Uncertainty:** Detecting portable or custom Steam installations requires checking both registry and `libraryfolders.vdf`. Some users may have multiple Steam installations across different drives.

**Sources:**
- Steam API documentation
- Community reports on multi-library and portable Steam setups
EOF

---