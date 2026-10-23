# Aethelgard Atlas — design decisions, discussion record, and implementation plan

**Status:** living design document  
**Last updated:** 2026-10-23  
**Applies to:** tools/aethelgard-atlas/index.html, docs/ATLAS_ROADMAP.md, future map-data schema and storage.

This document preserves the ideas agreed in conversation, distinguishes user requirements from implementation proposals, and records what is implemented versus what is still aspirational. It is not a claim that the prototype is already a full GIS.

## 1. Discussion record

The following user requests are copied verbatim from this conversation, including the original spelling.

> нужно сделать маштаб карты. Чтобы страны не были микроскопическими. но врядли мы сможем сделать как у гугл мэпс что чем ближе маштаб тем подробнее карта аж до карт городов. Во вторых сделать карту интересной за счет того, что она тоже будет как бы из будущего. То есть нужно придумать дизайн, интерфейс, стиль как будто из будущего. Представь как бы в далеком будущем рисовались бы границы? А рельнш? Или еще чтото.

> Нужно удобно сделать границы. Например, граница на самом деле определеяется на самом последнем слое. То есть на самом подробном, так во первых можно очень локально и точно менять границы, во вторых не получится как в azgaard где невохможно реально удобно менять границы государств

> если ты сможешь это реализовать, наверняка нужно будет какаято датабаза или олачное хранилище

> давай, во первых запиши все идеи внутрь, можешь даже скопироваь наши обсуждения дословно и добавить к ним свои окмментарии, во вторых можешь начать делать принимая во внимание все наши идеи

Ключевые формулировки из ответов ассистента, сохранённые дословно как рабочие правила:

> Граница государства должна быть не отдельным контуром, который приходится править на каждой карте, а геометрией, определённой на самом подробном уровне. Все остальные масштабы должны получать её из этой геометрии.

> Масштаб — это не только увеличение пикселей. При изменении масштаба система должна переключать наборы объектов, подписи и уровень детализации. Геометрия канона при этом не должна меняться.

> Не стоит считать, что каждая страна обязана иметь собственную независимую линию на каждом уровне детализации. Лучше хранить единую точную геометрию границ, а для отображения создавать упрощённые представления.

> Нам не обязательно сразу поднимать сервер или оплачивать облако. Сначала стоит укрепить геометрию и локальное сохранение; облако подключать, когда появится реальная потребность в синхронизации и совместной работе.

Earlier project context, summarized from the design discussion:
- The user wanted a small browser-based editor inspired by Google Maps for the fictional political geography, using Europe as a starting geography but allowing substantial alterations. The user explicitly did not want hand-drawn illustrative maps as the main solution.
- Aethelgard Atlas should ultimately support easy boundary editing, a continuous sense of scale, local details, and a visual language believable for a sophisticated cartographic system in 2147.
- We should prioritize practical map editing rather than only making a pretty mock-up.

## 2. Core principles

### 2.1 One authoritative geometry, many display scales

A political boundary must not be manually redrawn at each zoom level. The highest-detail geometry is the source of truth. Overview, regional, city, and local views are derived presentations of the same underlying geometry and linked feature data.

At small scales, lines and labels may be simplified for legibility. That simplification is a display operation only: it must not overwrite, delete, or weaken the authoritative geometry. When the map zooms back in, the editor reveals the detailed geometry that was already stored.

**Comment:** “The boundary is defined at the most detailed layer” is the design rule. It does not require every square kilometre of the world to be represented with millions of points. Add detail only where it is needed; store sparse geometry elsewhere. A local plan can have very fine detail while a remote wilderness remains generalized.

### 2.2 A shared political border is one object

Two neighbouring territories must not own independent copies of the same line. Otherwise, moving one copy creates gaps, overlaps, or contradictory borders.

The same canonical boundary geometry must be referenced by every territory that uses it. Editing a shared vertex or segment updates every adjacent territory. Boundary objects can carry metadata such as jurisdiction type, source, confidence, legal status, disputed status, and dates of validity.

**Comment:** A robust future data model will need topological validation as well as shared IDs. It must deal with junctions, multi-territory intersections, coastlines, enclaves, holes, and cases in which a political line does not follow a physical feature.

### 2.3 Scale must improve usability, not falsify geography

Small states should never be impossible to select. Provide “Fit selected territory”, search/focus, sensible padding, visible handles, and larger interaction hit areas. Do not distort canonical coordinates to make a country look bigger.

Continuous zoom alone is not equivalent to Google Maps-style detail. New detail only appears where a corresponding dataset or authored local plan exists. If a city or local plan has not been authored, say so rather than magnifying an empty map indefinitely.

### 2.4 The map should look like cartography from 2147

The target is a professional government/corporate geospatial workstation, not a neon cyberpunk game map.

Visual direction:
- Graphite and deep mineral backgrounds; subdued terrain hues; careful gold/amber accents for Aethelgard; distinct colours for other political entities.
- Use more than colour to distinguish lines: thickness, dash pattern, glow, labels, and contextual metadata can distinguish international, internal administrative, corporate-jurisdiction, demilitarized, and disputed borders.
- Relief should communicate actual landforms: elevation tinting, hillshade, contour lines, ridges, plateaus, valleys, river basins, glaciers, passes, coastlines, and engineered tunnels where appropriate.
- Smart cartographic labels, coordinate grid, scale bar, selected-feature glow, coordinate/elevation readout, layer provenance, date/version, and confidence indicators should add analytical value.
- Use effects sparingly. Legibility beats decoration.
- Plan two presentations: a dark operational screen and a light atlas/print mode.

### 2.5 Canon is editorial data, not an automatic side effect

Territories, cities, capitals, and border changes must be labelled as canon, proposal, or draft. New geometry must not be treated as approved canon merely because the editor can draw it. Important changes should eventually record author, time, rationale, source, and the canon decision that approved or rejected the change.

## 3. Scale and detail levels

1. **Political overview:** countries, alliances, major cities, key corridors and geopolitical blocs.
2. **Regional view:** international and administrative borders, terrain, rivers, settlements, mountain passes, transport corridors and disputed zones.
3. **City view:** districts, transit systems, tunnels, industrial areas, important buildings and municipal boundaries.
4. **Local plan:** buildings, compounds, fortifications, facilities, individual lots, and other fine-grained geometry where the author has created the plan.

The four levels are conceptual data/display levels, not four unrelated copies of the map. Each feature should keep a stable ID, and local plans should link to parent places. Derived generalization may create display geometry, but must not replace the original.

Quick navigation should include:
- Fit all / whole map
- Fit selected territory
- Search and focus a country or named feature
- Region / country / city / local-plan navigation when such data exists
- A clear “Detailed plan not created” state for missing local geometry

## 4. Boundary editing behaviour

### Expected interaction

- A dedicated boundary-edit mode separates editing from ordinary selection.
- Select a boundary line, inspect the territories that use it, and drag an existing vertex.
- Moving a shared vertex updates all adjacent territorial outlines because they reference the same geometry.
- Insert intermediate vertices along a selected boundary to create a more detailed local shape.
- Remove intermediate vertices when safe.
- Pan by dragging empty map space; zoom with the wheel or zoom controls.
- Provide undo/redo, autosave feedback, explicit project export, and a safe recovery path.
- Show errors or warnings for invalid geometry rather than silently accepting a broken outline.

### Topology guardrails

- A boundary junction shared by multiple boundary objects cannot be deleted as if it were an ordinary intermediate point. It needs a dedicated merge/re-node operation so the graph remains connected.
- Snapping and segment splitting must be explicit. A new line that crosses an existing line should eventually split and connect the topology where appropriate; it must not only look as if the lines touch.
- Validation should detect self-intersections, rings that do not close, missing references, duplicate edges, overlaps, gaps, sliver polygons, unclosed boundaries, and invalid holes.
- Boundary objects should be typed, for example international, administrative, corporate jurisdiction, disputed, demilitarized, or historical. Display style follows type/status, not colour alone.
- Historical or alternative borders should be versioned alternatives, not accidental duplicates that fight with the active geometry.

### Scope note

The current prototype implements vertex dragging and adding/removing some intermediate vertices. Deleting junction vertices, robust topology repair, snapping, arbitrary line intersections, and full geometry validation remain future work. The editor should state this limitation rather than silently corrupt shared topology.

## 5. Data architecture

### 5.1 Prototype schema: version 2

The current browser project now uses a topological representation:

- **vertices**: stable vertex IDs with x/y coordinates in the current schematic coordinate space.
- **boundaries**: stable boundary IDs with ordered vertex IDs and boundary metadata.
- **territories**: political metadata and ordered ring references. Each ring refers to shared boundary IDs plus a direction flag; the rendered polygon is assembled from those references.
- **cities**: named point features.
- **metadata**: project/schema information and update timestamp.

Legacy projects containing separate territory point arrays are migrated by detecting exact shared edges and reusing a common boundary object when endpoints match. This is a migration aid, not a complete topological repair algorithm.

Important caveat: the current map still uses a schematic x/y plane, not a proper geographic coordinate reference system. The GeoJSON export preserves the geometry shape in its current coordinates, but those x/y values are not WGS84 longitude/latitude. Treat it as a project interchange draft, not yet as ready-to-use geographic data for GIS software. A real projection and coordinate reference metadata are required before claiming geographic interoperability.

### 5.2 Local-first storage

Initial storage:
- IndexedDB is the primary browser-side project store.
- localStorage provides a fallback/backup copy for browsers or contexts where IndexedDB is unavailable.
- JSON project export/import provides portable backups between browsers and devices.
- GitHub stores application code, documentation, schema migrations, and reviewed canonical releases. It should not be the live database for frequent drag operations.

This design avoids requiring a cloud account or paid hosting just to edit the map.

### 5.3 Cloud and database path

When cross-device sync, multiple editors, larger datasets, permissions, or concurrent editing become necessary, migrate to:
- PostgreSQL with PostGIS for spatial storage and validation.
- A small application/API layer with authentication and permission checks.
- Optional managed hosting such as a managed PostgreSQL/PostGIS service; evaluate cost, data export, backups, and lock-in before choosing a provider.
- Conflict handling or collaborative editing only after stable geometry IDs, revisions, and transactions are defined.

Do not make cloud hosting the first milestone. It cannot compensate for a weak topology model.

### 5.4 Future data entities

A production schema will likely need:
- territories and political metadata
- vertices and canonical boundary geometries
- boundary-to-territory relationships
- map features and parent/child plan relationships
- geometry versions and historical validity dates
- provenance/source records and confidence
- editorial/canon status and approvals
- change log, authorship, revision IDs, and conflict information
- layers and coordinate reference system metadata

These are design targets, not all implemented tables.

## 6. Futuristic layers and visual information

Potential layers, ordered by practical value:
- Political boundaries and disputed claims
- Administrative boundaries
- Landform relief, elevation, slope shading, contour lines
- Rivers, lakes, coastlines, glaciers, passes, watersheds
- Transport and logistics corridors, high-speed lines, tunnel networks
- Major settlements, districts, infrastructure, industrial zones
- Corporate jurisdictions and special economic territories
- Historical snapshots (pre-/post-2089, and 2147)
- Source/confidence layer showing established geography, derived data, or author proposals
- Layer date, update state, and coordinate/elevation inspection

The “future” should come from information design and confident cartographic hierarchy, not from a sci-fi texture pasted over the map.

## 7. What is implemented in the prototype now

Current editor file: tools/aethelgard-atlas/index.html.

- Zoom via wheel and controls, pan by dragging empty map space.
- Fit all and fit selected territory, so small territories can be inspected without changing their canonical geometry.
- Boundary edit mode with visible vertices and draggable geometry.
- Boundary objects can be shared by territories when legacy edges match exactly; a shared boundary is rendered once and its vertices are the authoritative geometry for all territories referencing it.
- Insert an intermediate vertex along the selected boundary and remove some safe intermediate vertices.
- Create a new territory outline; exact existing boundary edges can be reused.
- Undo/redo for project edits.
- Browser autosave using IndexedDB plus localStorage fallback.
- Import/export project JSON and an initial GeoJSON-shaped export.
- Territory names, political type, capital, canon status, colour, and notes.
- Existing seed territories are schematic and provisional; they are not a declaration that names, borders, or capitals are canon.

## 8. Known limitations of this implementation

- The underlying map and relief are stylized schematic SVG artwork, not real Europe GIS data or a canonical terrain model.
- Zoom/pan is implemented, but there are not yet true separate regional/city/local-plan datasets or automatic data-driven level-of-detail switching.
- Shared boundaries are recognised for exact matching edges during legacy migration and edge creation. Arbitrary overlaps, partial overlaps, line intersections, auto-snapping, and full planar topology are not solved yet.
- Junction deletion is guarded; a proper merge/split tool is still needed.
- City points can be displayed but do not yet have a full drag/edit workflow.
- Undo/redo is in-memory for the current session; it is not a durable long-term revision history. Autosave is not cloud sync.
- The GeoJSON-shaped export uses schematic coordinates and should not be treated as geographically referenced longitude/latitude data.
- There is no multiuser locking, access control, cloud backup, or collaborative editing yet.

## 9. Implementation order

### P0 — protect the work and make editing dependable
1. Test browser storage, restore behaviour, import validation, undo/redo and reload persistence.
2. Add automated checks for schema and geometry references.
3. Add reliable vertex selection, insert/delete, pointer/touch handling, and feedback for unsafe operations.
4. Add topology diagnostics for missing references, invalid rings, self-intersections, gaps and overlaps.
5. Store app/schema version and provide migration logic for old project files.

### P1 — proper map authoring
1. Add safe vertex and boundary junction merge/split operations.
2. Add snapping, edge splitting, and shared-boundary creation for partially overlapping or intersecting edges.
3. Add editable cities and other feature types.
4. Add layers, filters, search, minimap, coordinate readout, scale bar, and light print/atlas mode.
5. Load real open geodata with license and provenance recorded; choose a geographic projection/CRS.
6. Separate physical geography from political geometry, then author the alternate 2147 geography.

### P2 — detail and long-term canon
1. Add linked city/local plans and scale-dependent label/feature visibility.
2. Add stable feature IDs, source/confidence fields, validity dates, historical border snapshots and change comparison.
3. Add SVG/PNG print output and correctly referenced GeoJSON.
4. Add persistent revision history and reviewed canonical releases.
5. Only then choose PostGIS hosting and implement account-based cross-device sync or collaborative editing.

## 10. Acceptance criteria for a dependable first release

A user can select any territory even if small, zoom and pan predictably, fit the selected territory, drag a shared border without creating a gap between adjacent territories, insert a detailed local vertex, undo/redo changes, reload the page without losing work, export/import a project backup, and see whether a territory is canon, proposal, or draft. The app must clearly disclose when a displayed map or export is only schematic.
