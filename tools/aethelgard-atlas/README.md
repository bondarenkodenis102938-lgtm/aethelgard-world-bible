# Aethelgard Atlas · Victorian Terrain Lab

The active map is a terrain-first geographic workbench. The earlier schematic political-boundary editor is archived as `legacy-political-editor.html`.

## Development status

**The map is not being published yet.** The active file is developed and tested locally before any optional deployment.

The default **Victorian Atlas** appearance is a first-pass cartographic treatment layered over OpenFreeMap's contemporary vector basemap: muted paper, sepia/olive lines, restrained water colours and a subtle print-like tint. It changes the way existing data is rendered; it is not a historical 19th-century dataset, and it does not claim that modern streets or borders are canon for Aethelgard in 2147.

## Run locally

Serve this folder with a local static HTTP server and open `index.html` in a modern browser with internet access. MapLibre is loaded as an ES module from a CDN. Map tiles come from OpenFreeMap, elevation from AWS Terrain/Terrarium tiles, and place search from Photon. These public services do not guarantee availability.

## Current controls

- Mouse wheel / map controls: continuous zoom; drag to pan.
- Top shortcuts: Europe, Alps, Rhine Valley, city and local block. The map's navigation is constrained to Europe.
- Relief controls: hillshade, 3D terrain and experimental 3D buildings.
- Basemap appearance: Victorian Atlas, dark and modern light.
- Basemap layer groups: land cover/parks, water, administrative boundaries, roads, rail/transit, building footprints, POIs and labels can be toggled separately. Layer matching follows the current provider style's layer names, so some provider-specific items may be grouped imperfectly.
- Search: look up towns and streets within the European bounding area.
- Click a named map feature to inspect the attributes exposed by the source.

## Author-created GeoJSON overlays

Use **Import GeoJSON** to add one or more `.geojson` or compatible `.json` files. Each imported file becomes its own overlay above the basemap. Supported input forms are a GeoJSON `FeatureCollection`, `Feature`, or geometry object.

For each overlay, the UI provides independent visibility, export and removal controls. **Export all** combines visible and hidden overlays into one GeoJSON file, preserving an `_atlas_layer` property to identify each feature's original layer. Imported data is persisted to local browser storage, so it is available again in the same browser profile on this machine. Export important work as a backup; browser storage is not cloud storage and may be cleared or run out of space.

A ready-to-import example is available at [`examples/sample-aethelgard-proposals.geojson`](examples/sample-aethelgard-proposals.geojson). It contains an illustrative study polygon, a conceptual transit line and a test marker near the Vaduz reference area. These are **fictional demonstration features, not canon**.

These first overlays are for adding reference geometry and future Aethelgard concepts without modifying the external basemap. They do **not yet** provide drawing/editing of individual vertices, moving roads/buildings, editing the terrain mesh, topology validation or a shared project database. Those are later development stages.

## Important limits

- The base map remains contemporary real-world geography and OSM-derived data, not an authoritative map of Aethelgard 2147.
- Local detail depends on OpenStreetMap coverage and the published vector tiles. High zoom does not invent unmapped roads or building footprints.
- The elevation tiles have a lower native maximum zoom than the vector map; at very close zooms the terrain may be overzoomed.
- Victorian styling recolours provider layers when their style schema permits it and applies a mild canvas tint. It is a starting aesthetic, not yet engraved contour cartography.
- Layer grouping is inferred from provider layer/source-layer names. If a new provider style uses different names, some features may not follow the expected toggle.
- Public services are external dependencies; keep attribution visible and follow their terms.

## Publication

`.github/workflows/deploy-atlas.yml` is manual-only and is not triggered by ordinary commits. Do not publish until explicitly requested.

See `../../docs/ATLAS_DATA_SOURCES.md`, `../../docs/ATLAS_ROADMAP.md` and `../../docs/ATLAS_DESIGN_DECISIONS.md` for sources, constraints and upcoming work.
