# Aethelgard Atlas · Terrain Lab

This is the active, terrain-first map prototype. The older schematic political-boundary editor is preserved as legacy-political-editor.html.

## Run

Open index.html in a modern browser with an internet connection. The page loads MapLibre GL JS from a CDN, vector map tiles from OpenFreeMap, elevation tiles from AWS Open Data and place search from Photon. It is a static client-side app; there is no server or API key required for the current public sources.

A GitHub Pages workflow is included at .github/workflows/deploy-atlas.yml. The first workflow attempt cannot create the Pages site because Pages is not enabled yet. To publish a live preview, open repository Settings → Pages → Build and deployment and set Source to “GitHub Actions”; then rerun “Deploy Aethelgard Atlas” in Actions. The expected URL is https://bondarenkodenis102938-lgtm.github.io/aethelgard-world-bible/ once the deployment succeeds.

## Controls

- Mouse wheel or map zoom controls: continuous zoom.
- Drag empty map area: pan.
- Top scale shortcuts move outward from local block to city, Rhine Valley, Alps and the European overview.
- Right-side checkboxes: hillshade, 3D terrain, optional 3D buildings, roads, rail/transit, building outlines and labels.
- Search field: find a town, street or place; the request is sent only when the search form is submitted.
- Click a named map feature to inspect its available attributes.

## Important limits

- The current base layer is modern real-world geography around the former Liechtenstein reference region; it is not Aethelgard canon or a future-accurate map.
- Local detail depends on the completeness of OpenStreetMap and the public vector tiles. Higher zoom does not conjure up unmapped roads or buildings.
- Height tiles have lower native maximum zoom than the vector map. Very detailed terrain needs a more local licensed/open DEM dataset.
- The prototype does not yet let the user edit roads, buildings or relief. Custom Aethelgard layers and political boundaries are later tasks.
- Public services are external dependencies without a guaranteed service level; source attribution is shown in the application.

See ../../docs/ATLAS_DATA_SOURCES.md and ../../docs/ATLAS_ROADMAP.md for source, licence, limitation and development details.
