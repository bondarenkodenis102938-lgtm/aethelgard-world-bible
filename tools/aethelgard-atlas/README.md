# Aethelgard Atlas · Terrain Lab

This is the active, terrain-first map prototype. The older schematic political-boundary editor is preserved as legacy-political-editor.html.

## Development status

**The map is not being published yet.** Work is continuing on the geographic foundation and scale/detail model before we need a live website.

The MAP ENGINE UNAVAILABLE message appears when the external MapLibre GL JS script could not load in the browser. The current prototype loads MapLibre from a CDN, map tiles from OpenFreeMap, elevation tiles from AWS Open Data and place search from Photon. It needs an internet connection to display real tiles. This runtime dependency will be revisited during browser testing; it does not block the current data-model and design work.

## Run locally for development

Open index.html in a modern browser with internet access. No backend or API key is required for the configured public sources, but their availability is not guaranteed.

## Optional future publication

The file .github/workflows/deploy-atlas.yml only runs when manually started from GitHub Actions. It is intentionally not triggered by ordinary commits. When a live preview is explicitly wanted, enable GitHub Pages in repository Settings → Pages and choose “GitHub Actions” as the source, then run the workflow.

## Current controls

- Mouse wheel or map zoom controls: continuous zoom.
- Drag the map: pan.
- Top scale shortcuts move outward from local block to city, Rhine Valley, Alps and European overview.
- Right-side controls: hillshade, 3D terrain, optional 3D buildings, roads, rail/transit, building outlines and labels.
- Search field: find a town, street or place; requests are sent on form submit.
- Click a named map feature to inspect attributes available from the source.

## Important limits

- The basemap is contemporary real-world geography around the former Liechtenstein reference region; it is not Aethelgard canon or a future-accurate map.
- Local detail depends on OpenStreetMap coverage and the public vector tiles. Higher zoom does not create unmapped roads or buildings.
- Height tiles have a lower native maximum zoom than the vector map. Very detailed terrain needs a more local licensed/open DEM dataset.
- The prototype does not yet allow editing roads, buildings or relief. Custom Aethelgard layers and political boundaries are later tasks.
- Public services are external dependencies; source attribution is shown in the application.

See ../../docs/ATLAS_DATA_SOURCES.md and ../../docs/ATLAS_ROADMAP.md for data-source, license, limitation and development details.
