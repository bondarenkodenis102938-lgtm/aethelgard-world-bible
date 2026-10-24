# Aethelgard Atlas — design decisions, discussion record, and implementation plan

**Status:** living design document  
**Last updated:** 2026-10-24  
**Applies to:** tools/aethelgard-atlas/index.html, docs/ATLAS_ROADMAP.md, future map-data schema and storage.

This document preserves the ideas agreed in conversation, distinguishes user requirements from implementation proposals, and records what is implemented versus what is still aspirational. It is not a claim that the prototype is already a full GIS.

**Priority decision (24 October 2026):** the active application is now a real-data, terrain-first map viewer. Political borders and the previous drawing prototype are deliberately parked until the base map, terrain, scale behaviour and local feature detail are satisfactory. The former boundary editor is archived at tools/aethelgard-atlas/legacy-political-editor.html.

## 1. Discussion record

The following user requests are copied verbatim from this conversation, including the original spelling.

> нужно сделать маштаб карты. Чтобы страны не были микроскопическими. но врядли мы сможем сделать как у гугл мэпс что чем ближе маштаб тем подробнее карта аж до карт городов. Во вторых сделать карту интересной за счет того, что она тоже будет как бы из будущего. То есть нужно придумать дизайн, интерфейс, стиль как будто из будущего. Представь как бы в далеком будущем рисовались бы границы? А рельнш? Или еще чтото.

> Нужно удобно сделать границы. Например, граница на самом деле определеяется на самом последнем слое. То есть на самом подробном, так во первых можно очень локально и точно менять границы, во вторых не получится как в azgaard где невохможно реально удобно менять границы государств

> если ты сможешь это реализовать, наверняка нужно будет какаято датабаза или олачное хранилище

> давай, во первых запиши все идеи внутрь, можешь даже скопироваь наши обсуждения дословно и добавить к ним свои окмментарии, во вторых можешь начать делать принимая во внимание все наши идеи

> так, да, изза того, что границы теперь стали парочкой точек они очень квадртаный и точки будут нагружать очень систему если они будет много и будут подробно изображать границу. Так что нужно подумать как сделать это иначе, может быть, нам просто рисаовать от руки? или два режима один редактирвани а второй скорее обычная карта

> братан я думаю нам снчала не нужно заморчиваться с граниами государств и отрисовки политической карты. Я думаю сначаала нам нужно определить маштаб насколько близко можно приблизить каарту, сделать рельеф и начать именно с локального плана постепенно повыщая детализацию вверх, то есть сначала нам нужно детализироватьь и сделать макет карты, а потом с учетом всего. Выше мы выстроили карту на 4 уровня, думаю сначала лоичнее всего сделать карту картой, а потом уже налаживать на нее остальное. Если знаешь как лучше сделать делай, можешь пользоватья публичными данными, можешь также воспольщоваться опытом гугл карт как они это делают. То есть если маштаб похволяет я хочу видеть не просто точку, а реальный гррод, дороги железные дороги и тд. Жальше можно уже долелать. Нам не обязательно отрисовыватьвсю эвропу, пока только самое важное

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

- Explicit **Map mode** and **Edit mode**. Map mode is for reading the atlas, selecting territories, zooming and panning; it does not expose edit handles. Edit mode reveals handles only for the selected boundary, not for every point on the map.
- Select a boundary line, inspect which territories refer to it, and drag an existing vertex. A shared vertex updates all outlines that reference it.
- Insert intermediate vertices along a selected boundary and remove intermediate vertices when safe.
- Draw a new territory freehand with a mouse or stylus in one continuous stroke, then finish the contour. The editor samples the stroke rather than storing every pointer event, simplifies it, and groups it into a small number of curved boundary sections.
- The display uses smooth SVG curves for multi-vertex boundary sections. The authoritative vertex chain remains saved separately; viewing/zooming does not replace the original geometry.
- Zoom level controls display generalization. At overview scale, tiny deviations are removed from the rendered path; closer in, more of the stored detail is shown. Display simplification must not mutate stored geometry.
- Pan by dragging the map and zoom with the wheel or controls. Map mode must suppress territory selection after a real drag, while a simple click can still select a territory.
- Provide undo/redo, autosave feedback, explicit project export, and a safe recovery path. Invalid or risky geometry operations should warn rather than silently corrupt topology.

### Stroke and rendering budget

- Freehand input is sampled at a screen-space interval rather than on every raw pointer event. The current prototype samples at about 2.4 CSS pixels, then applies a closed-ring Ramer–Douglas–Peucker simplification using a tolerance related to current zoom.
- The simplified contour is split at prominent turning points into curved boundary sections, capped at 18 sections per new contour in the prototype. Many sample points can therefore be held by a few SVG path elements instead of producing one DOM element for each point.
- The map renderer simplifies long polylines for display at roughly 1.35 screen pixels of tolerance, and caches simplified coordinates for the duration of a render pass. Zoom changes cause the display geometry to be regenerated from the stored geometry.
- Smooth curves are a rendering interpolation over the stored polyline, not yet a persistent spline geometry format. The current GeoJSON export follows the stored straight segments and may not exactly reproduce the on-screen interpolated curve. A future geographic export should densify/render the interpolated curve or adopt an explicitly shared curve model.
- Do not make every border permanently dense. Keep detail where the author drew it; when a scale cannot resolve that detail, display generalized geometry instead of creating unnecessary SVG objects.

### Topology guardrails

- A boundary junction shared by multiple boundary objects cannot be deleted as if it were an ordinary intermediate point. It needs a dedicated merge/re-node operation so the graph remains connected.
- Snapping and segment splitting must be explicit. A new line that crosses an existing line should eventually split and connect the topology where appropriate; it must not only look as if the lines touch.
- Validation should detect self-intersections, rings that do not close, missing references, duplicate edges, overlaps, gaps, sliver polygons, unclosed boundaries, and invalid holes.
- Boundary objects should be typed, for example international, administrative, corporate jurisdiction, disputed, demilitarized, or historical. Display style follows type/status, not colour alone.
- Historical or alternative borders should be versioned alternatives, not accidental duplicates that fight with the active geometry.

### Scope note

The current prototype now includes separate map/edit modes, one-stroke freehand contour creation, stroke simplification, smoothed rendering for multi-vertex boundary sections, view-dependent display simplification, a render-pass geometry cache, and vertex handles only on the selected boundary. It also supports freehand redraw of a selected boundary chain, preserving its endpoint vertices so all territories using that boundary update together. Redrawing is blocked if a section contains interior junctions shared with other boundary objects. Deleting junction vertices, robust topology repair, snapping, arbitrary intersections, full geometry validation, and replacing an entire territory ring remain future work. The editor should state these limitations rather than silently corrupt shared topology.

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

## 7. What is implemented in the active prototype now

Active entry point: tools/aethelgard-atlas/index.html. Previous political sketch: tools/aethelgard-atlas/legacy-political-editor.html.

- MapLibre GL JS renders a real interactive base map from OpenFreeMap / OpenStreetMap vector tiles rather than the old illustrative SVG continent.
- The initial camera opens at city scale around the former Liechtenstein reference area, so the workflow starts close and zooms outward as needed.
- Camera bookmarks move from Central Europe to the Alps, Rhine Valley, city and local street/block scale. Zooming remains continuous between these bookmarks.
- The basemap includes available real-world roads, railways/transit, named places, water features and building footprints from public vector data.
- An AWS Terrarium elevation source is used for hillshade and MapLibre 3D terrain.
- Layer switches control roads, rail/transit, building outlines, labels, hillshade, 3D terrain and (when the active style exposes building vector data) an experimental 3D building layer.
- Search uses Photon over OpenStreetMap data on submit, with a basic request pause; clicking map features shows the attributes available in the tiles.
- Dark and light basemap styles are available; map position, zoom and a few style options are saved in localStorage.
- Visible provider/data attribution is included. A separate source register lives in docs/ATLAS_DATA_SOURCES.md.

## 8. Known limitations of this implementation

- The basemap is contemporary real-world geography around the former Liechtenstein area. It is a reference substrate, not a finished map of Aethelgard in 2147 and not a claim that modern borders, roads or buildings survived unchanged.
- Detailed streets, railways and building footprints depend on OpenStreetMap coverage and the public vector-tile provider. A high zoom number does not guarantee that every alley or building plan exists.
- The public terrain source has a lower maximum native zoom than the vector map; close-in relief may be overzoomed. Very precise local elevation or engineering plans will require better regional DEM/LiDAR data if available and appropriately licensed.
- The 3D-building layer is experimental: footprint coverage and height attributes are uneven, so it can be empty in some styles/areas or estimate height for buildings lacking metadata.
- The application is a map viewer, not yet a GIS editor for roads, buildings or terrain. It does not yet allow creating custom local features on top of the basemap.
- The app loads MapLibre, vector tiles, elevation and place-search from external public services. An internet connection is required; these public services can change and generally do not guarantee an SLA.
- The Photon public demo can throttle requests; search is deliberately user-triggered rather than search-as-you-type.
- No offline mode, custom regional tile cache, own geodata backend, cloud sync, user accounts or collaborative editing exists yet.
- The previous polygon/boundary editor remains in legacy-political-editor.html. Its point-based topology, freehand curves and schematic coordinates are not part of the active map-viewer workflow and should not be treated as geographic data.

## 9. Implementation order

### P0 — make the base map dependable
1. Verify the map in a real browser and check loading/error behaviour for MapLibre, OpenFreeMap and elevation tiles.
2. Test camera presets, zoom limits, clicking features, style switching, 2D/3D terrain, and each layer toggle.
3. Adjust default zoom and camera so the first view shows useful local detail without hiding the wider Alpine relief.
4. Confirm attribution and public-service usage constraints remain visible and documented.

### P1 — make the priority region truly useful
1. Compare real data coverage at city, street and local-block scales around the Aethelgard reference region.
2. Choose a small number of high-value local zones instead of building all of Europe in detail.
3. Add supplemental authored GeoJSON/GeoPackage layers where open data is incomplete; keep them separate from the provider's base map.
4. Add more precise regional elevation sources where licensing and access permit.
5. Add a layer inspector/source-confidence field so real-world reference data can be distinguished from author-created 2147 geography.

### P2 — build the fictional geography on top
1. Define the Aethelgard 2147 physical changes: terrain, valleys, waterways, passes and engineered infrastructure, without immediately forcing political borders into the model.
2. Store author-created features with stable IDs, provenance and a canon/proposal/draft status.
3. Add editable political boundaries only after the physical/map foundation is good enough to work on.
4. Bring back the shared-topology editor as a dedicated edit mode or module, using the map's real coordinate reference system.
5. Consider local storage databases or PostGIS only when custom data volume, offline needs or collaboration justifies them.

## 10. Acceptance criteria for a dependable first release

A user can open the active map with an internet connection, see real roads and railways in the priority region, switch relief and map layers, navigate smoothly from regional to local zoom, search for a place, inspect available feature attributes, and distinguish the contemporary reference map from future Aethelgard canon. The app must clearly disclose missing map detail and data-source limits. Political editing is a later milestone, not a requirement for this first terrain-first release.


## 11. Victorian cartography and first custom-layer workflow (24 October 2026)

The active prototype now has a first-pass **Victorian Atlas** appearance. It starts from the public OpenFreeMap Liberty vector style, then repaints accessible vector-layer colours into a paper, sepia, muted-olive and stone palette; the canvas receives a restrained tint/print texture. The dark and modern light styles remain available. This is a visual treatment of contemporary OSM-derived data, not a 19th-century source dataset or proof that current objects exist unchanged in Aethelgard 2147.

Basemap layer visibility is split into inferred groups: land cover/use, water, administrative boundaries, roads, rail/transit, buildings, POIs and labels, plus the existing hillshade / 3D terrain / experimental 3D buildings controls. Grouping depends on provider layer and source-layer names and needs browser verification.

The prototype can now import each GeoJSON file as an independent user layer; each can be toggled, exported, or removed. The combined export marks each feature with an `_atlas_layer` property. Imported data and visibility persist in browser localStorage. This gives us an initial separation between provider basemap and Aethelgard-authored geography. It is not yet direct vertex editing, drawing, topological validation, terrain editing, or shared/cloud storage. Export important work as a backup.

The next practical test is local browser validation: confirm the Victorian paint changes remain readable, each basemap toggle affects the intended group, and a small GeoJSON test file can be imported, hidden, exported, restored after refresh, and deleted. Do not publish automatically; the deployment workflow remains manual-only.
