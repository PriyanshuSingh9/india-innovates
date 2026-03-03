# Geospatial Frontend

> Covers: F04 MapLibre GL Integration · F05 Deck.gl Layer System · F06 India Static Geo Layers · F07 Regional Preset System · F13 Map Event Markers · F30 Instability Heatmap Layer

---

## F04 — MapLibre GL Integration

### What Is Being Built

The core map component of BIE — an interactive, GPU-accelerated, full-screen map rendered with MapLibre GL JS, centered on the Indian subcontinent. This is the primary visual canvas on which all intelligence data is displayed.

### Why It Is Needed

BIE is fundamentally a **geospatial** intelligence platform. Every event, threat, vessel position, and instability score has a geographic component. Without a performant map:

- Events are just a list of text items with no spatial awareness
- Analysts can't see geographic clustering (e.g., three military events near Aksai Chin in one day)
- There's no way to visualize the LAC, LOC, IOR theater, or infrastructure proximity
- The demo's opening moment ("Show India-centered dark map. Toggle LAC and LOC layers. Zoom into Aksai Chin.") is impossible

MapLibre GL JS is chosen over Google Maps or Mapbox because:
- **Open source** — no API key costs or usage limits
- **Vector tiles** — smooth zooming, custom styling, dark mode
- **WebGL rendering** — handles thousands of markers without jank
- **Deck.gl compatible** — Deck.gl overlays work natively with MapLibre

### What Use It Provides

- **India-centric dark-themed map** — immediate visual impact at demo open
- **Smooth vector tile rendering** — zoom from country level to street level seamlessly
- **Custom styling** — dark base map with bright event markers for high contrast
- **Interactive controls** — pan, zoom, rotate, pitch (3D tilt)
- **Programmatic camera** — fly-to animations when clicking events or selecting regional presets

### How It Is Built

1. **Install dependencies**:
   ```bash
   cd frontend
   npm install maplibre-gl
   ```

2. **Map component** — `frontend/app/components/Map.js`:
   ```jsx
   "use client";
   import { useEffect, useRef } from "react";
   import maplibregl from "maplibre-gl";
   import "maplibre-gl/dist/maplibre-gl.css";

   export default function Map({ onMapReady }) {
     const mapContainer = useRef(null);
     const map = useRef(null);

     useEffect(() => {
       map.current = new maplibregl.Map({
         container: mapContainer.current,
         style: "https://basemaps.cartocdn.com/gl/dark-matter-gl-style/style.json",
         center: [78.9629, 22.5937],  // Center of India
         zoom: 4.5,
         pitch: 0,
         bearing: 0,
       });

       map.current.addControl(new maplibregl.NavigationControl(), "top-right");

       map.current.on("load", () => {
         onMapReady?.(map.current);
       });

       return () => map.current?.remove();
     }, []);

     return <div ref={mapContainer} className="w-full h-full absolute inset-0" />;
   }
   ```

3. **Dark theme**: use CartoDB Dark Matter or a custom MapLibre style JSON with muted landmass colors, dark water, and minimal labels.

4. **Camera presets**: expose a `flyTo()` wrapper for animated camera transitions:
   ```javascript
   function flyToRegion(map, { center, zoom, bearing = 0, pitch = 0 }) {
     map.flyTo({ center, zoom, bearing, pitch, duration: 2000, essential: true });
   }
   ```

---

## F05 — Deck.gl Layer System

### What Is Being Built

A layer management system using Deck.gl that overlays data visualizations on top of the MapLibre base map. Deck.gl provides high-performance WebGL layers for scatterplots (event markers), heatmaps (instability), arcs (connections), and icon layers (infrastructure).

### Why It Is Needed

MapLibre handles the base map tiles, but BIE needs to render **dynamic data layers** that update in real-time:

- **Thousands of event markers** positioned at precise coordinates
- **Heatmap overlays** showing instability intensity across regions
- **Arc lines** showing connections between entities in the knowledge graph
- **Icon markers** for military bases, ports, airfields

Native MapLibre markers (DOM elements) don't scale beyond ~100 markers without jank. Deck.gl renders all markers as WebGL primitives on the GPU, handling 10,000+ points at 60fps.

### What Use It Provides

- **ScatterplotLayer** — event markers colored by threat level (green → yellow → red)
- **HeatmapLayer** — instability index visualization as a continuous color field
- **IconLayer** — infrastructure icons (military bases, ports, airfields)
- **ArcLayer** — entity relationship arcs for knowledge graph visualization
- **Layer toggle** — users can show/hide individual layers via a control panel
- **Performance** — GPU-rendered, handles large datasets without framerate drops

### How It Is Built

1. **Install**:
   ```bash
   npm install @deck.gl/core @deck.gl/layers @deck.gl/mapbox
   ```

2. **Layer manager** — `frontend/app/components/LayerManager.js`:
   ```jsx
   import { MapboxOverlay } from "@deck.gl/mapbox";
   import { ScatterplotLayer, IconLayer, ArcLayer } from "@deck.gl/layers";
   import { HeatmapLayer } from "@deck.gl/aggregation-layers";

   export function createOverlay(layers) {
     return new MapboxOverlay({
       interleaved: true,
       layers,
     });
   }

   export function createEventLayer(events) {
     return new ScatterplotLayer({
       id: "events",
       data: events,
       getPosition: (d) => [d.location.lng, d.location.lat],
       getRadius: (d) => threatRadius(d.threat_level),
       getFillColor: (d) => threatColor(d.threat_level),
       pickable: true,
       radiusMinPixels: 4,
       radiusMaxPixels: 20,
     });
   }
   ```

3. **Threat-level color mapping**:
   ```javascript
   const THREAT_COLORS = {
     LOW: [46, 204, 113, 180],       // Green
     MEDIUM: [241, 196, 15, 200],    // Yellow
     HIGH: [231, 76, 60, 220],       // Red
     CRITICAL: [192, 57, 43, 255],   // Dark Red, pulsing
   };
   ```

4. **Layer toggle panel**: a floating control with checkboxes for each layer (Events, Heatmap, Infrastructure, Connections). Toggle updates the Deck.gl overlay's `layers` array.

5. **Performance cap**: as per the risk register, cap ScatterplotLayer at 500 visible markers and disable HeatmapLayer on low-GPU devices.

---

## F06 — India Static Geo Layers

### What Is Being Built

GeoJSON-based boundary layers showing India's critical geopolitical lines: the Line of Actual Control (LAC), Line of Control (LOC), international borders, state boundaries, union territory boundaries, and key strategic zones (Aksai Chin, Arunachal Pradesh, Siachen, IOR chokepoints).

### Why It Is Needed

Intelligence events are meaningless without geographic context. An event near "Galwan Valley" means nothing on a blank map. But overlay the LAC line, and suddenly the event is visually positioned within meters of the disputed border — the significance is immediately obvious.

The demo script requires: *"Show India-centered dark map. Toggle LAC and LOC layers. Zoom into Aksai Chin."* This feature makes that moment possible.

### What Use It Provides

- **LAC line** — the de facto China-India border, drawn from published coordinates
- **LOC line** — the India-Pakistan de facto border in Kashmir
- **International borders** — India's recognized international boundaries
- **State boundaries** — all 28 states and 8 union territories
- **Strategic zones** — highlighted areas for Aksai Chin, Arunachal Pradesh, Doklam, Siachen Glacier
- **IOR chokepoints** — Strait of Malacca, Strait of Hormuz, Bab el-Mandeb
- **Toggle control** — each layer can be independently shown/hidden

### How It Is Built

1. **GeoJSON data files** in `data/geo/`:
   ```
   data/geo/
   ├── lac.geojson
   ├── loc.geojson
   ├── india_borders.geojson
   ├── india_states.geojson
   ├── strategic_zones.geojson
   └── ior_chokepoints.geojson
   ```

2. **Source the GeoJSON**: use Natural Earth Data (public domain) for country/state borders. LAC and LOC lines from published academic/government sources. Strategic zones are manually defined polygons.

3. **MapLibre layer addition**:
   ```javascript
   map.addSource("lac", { type: "geojson", data: "/geo/lac.geojson" });
   map.addLayer({
     id: "lac-line",
     type: "line",
     source: "lac",
     paint: {
       "line-color": "#e74c3c",
       "line-width": 2,
       "line-dasharray": [4, 2],
     },
   });
   ```

4. **Layer styling**: dashed red for LAC, dashed orange for LOC, solid gray for state borders, highlighted fill for strategic zones.

5. **Load on demand**: GeoJSON files are loaded after the map initializes, not bundled in the JS build.

---

## F07 — Regional Preset System

### What Is Being Built

A set of one-click camera presets that instantly fly the map to strategically important regions. Each preset defines a center coordinate, zoom level, pitch, and bearing optimized for that region's geography.

### Why It Is Needed

India is a vast country. An analyst monitoring the LAC needs a different view than one watching the Indian Ocean. Manually panning and zooming to each region wastes time and breaks the demo flow. Presets provide instant, cinematic transitions to key areas.

The demo script jumps between Aksai Chin and the IOR theater — presets make these transitions smooth 2-second animations instead of awkward manual panning.

### What Use It Provides

- **One-click navigation** to key strategic regions
- **Animated fly-to** transitions (2-second duration) for cinematic feel
- **Consistent framing** — every team member and every demo shows the same view of each region
- **Region-filtered data** — selecting a preset can also filter events/markers to that region

### How It Is Built

1. **Preset definitions**:
   ```javascript
   export const REGIONAL_PRESETS = {
     INDIA_OVERVIEW: {
       label: "India Overview",
       center: [78.9629, 22.5937],
       zoom: 4.5,
       pitch: 0,
       bearing: 0,
     },
     LAC_WEST: {
       label: "LAC — Aksai Chin",
       center: [78.5, 35.0],
       zoom: 7,
       pitch: 45,
       bearing: -20,
     },
     LAC_EAST: {
       label: "LAC — Arunachal",
       center: [93.5, 28.0],
       zoom: 7,
       pitch: 30,
       bearing: 10,
     },
     LOC_KASHMIR: {
       label: "LOC — Kashmir",
       center: [74.8, 34.1],
       zoom: 7.5,
       pitch: 30,
       bearing: 0,
     },
     SIACHEN: {
       label: "Siachen Glacier",
       center: [77.1, 35.4],
       zoom: 9,
       pitch: 60,
       bearing: -30,
     },
     IOR_OVERVIEW: {
       label: "Indian Ocean Region",
       center: [73.0, 5.0],
       zoom: 3.5,
       pitch: 0,
       bearing: 0,
     },
     STRAIT_MALACCA: {
       label: "Strait of Malacca",
       center: [101.5, 2.5],
       zoom: 6,
       pitch: 0,
       bearing: 0,
     },
     NORTHEAST: {
       label: "Northeast India",
       center: [93.0, 26.0],
       zoom: 6.5,
       pitch: 20,
       bearing: 0,
     },
   };
   ```

2. **Preset selector UI**: a dropdown or button row in the map controls area. Clicking a preset calls `map.flyTo(preset)`.

3. **URL sync**: the active preset is reflected in the URL hash (`#region=LAC_WEST`) so links can be shared.

---

## F13 — Map Event Markers

### What Is Being Built

Interactive markers on the map representing individual intelligence events. Each marker is a Deck.gl ScatterplotLayer point, colored and sized by threat level, with click-to-open detail panels showing extracted entities, source links, and proximity analysis.

### Why It Is Needed

The map needs to show *where* things are happening. The demo script requires: *"A live news item arrives in the feed. Click it. Map flies to the event location."* and *"Click the event marker. Show EventDetailPanel with extracted entities and proximity infrastructure."*

Markers are the visual bridge between the news feed (textual) and the map (spatial). Without them, the map is just a pretty background with no intelligence on it.

### What Use It Provides

- **Spatial awareness** — see all events positioned on the map simultaneously
- **Threat visualization** — color coding (green/yellow/red) instantly communicates severity
- **Click interaction** — clicking a marker opens a detail panel with full event information
- **Hover tooltip** — hovering shows title and threat level without clicking
- **Animation** — new events appear with a brief pulse animation; CRITICAL events pulse continuously
- **Clustering** — when zoomed out, nearby events cluster into a single marker showing count

### How It Is Built

1. **Event layer** (built in F05) is populated with real event data from `GET /api/events?bbox=...`

2. **Click handler**:
   ```javascript
   const eventLayer = new ScatterplotLayer({
     id: "events",
     data: events,
     getPosition: (d) => [d.location.lng, d.location.lat],
     getFillColor: (d) => THREAT_COLORS[d.threat_level],
     pickable: true,
     onClick: ({ object }) => {
       setSelectedEvent(object);
       map.flyTo({ center: [object.location.lng, object.location.lat], zoom: 10 });
     },
     onHover: ({ object, x, y }) => {
       setTooltip(object ? { object, x, y } : null);
     },
   });
   ```

3. **EventDetailPanel** — a slide-out panel showing:
   - Event title and summary
   - Source and timestamp
   - Extracted entities (people, organizations, locations)
   - Threat level badge
   - Infrastructure proximity list (military bases within 50km, etc.)
   - Link to original source

4. **Real-time updates**: new events from the SSE stream (`/api/events/stream`) are appended to the events array, triggering a layer re-render.

5. **Performance**: viewport-based filtering — only request events within the current map bounding box. Cap at 500 visible markers.

---

## F30 — Instability Heatmap Layer

### What Is Being Built

A continuous color heatmap overlay on the map showing the **India Instability Index (III)** — a per-region score from 0.0 (stable) to 1.0 (highly unstable). High-instability regions glow red/orange; stable regions are transparent. The heatmap updates as new events are ingested and scores recalculated.

### Why It Is Needed

Individual event markers show *what* happened. The instability heatmap shows *where tensions are building*. A cluster of LOW-threat events in a region might individually seem harmless, but the heatmap reveals that the cumulative instability score is rising — a pattern invisible from markers alone.

The demo script includes: *"Convergence alert fires near Aksai Chin. Posture panel escalates to HIGH."* The heatmap makes this visually dramatic — the region glows brighter as instability increases.

### What Use It Provides

- **Regional instability at a glance** — one look at the map shows where India's pressure points are
- **Temporal awareness** — scores update over time, showing whether regions are stabilizing or escalating
- **Anomaly highlighting** — sudden score spikes create visible "hot spots" on the map
- **Layered with markers** — heatmap provides context behind individual events

### How It Is Built

1. **Data source**: `GET /api/instability` returns per-region scores:
   ```json
   [
     { "region": "LAC_WEST", "score": 0.75, "lat": 35.0, "lng": 78.5 },
     { "region": "KASHMIR", "score": 0.82, "lat": 34.1, "lng": 74.8 },
     { "region": "NORTHEAST", "score": 0.45, "lat": 26.0, "lng": 93.0 }
   ]
   ```

2. **Deck.gl HeatmapLayer**:
   ```javascript
   import { HeatmapLayer } from "@deck.gl/aggregation-layers";

   const heatmapLayer = new HeatmapLayer({
     id: "instability-heatmap",
     data: instabilityScores,
     getPosition: (d) => [d.lng, d.lat],
     getWeight: (d) => d.score,
     radiusPixels: 80,
     intensity: 1.5,
     threshold: 0.1,
     colorRange: [
       [0, 0, 0, 0],         // Transparent (stable)
       [255, 255, 0, 100],    // Yellow
       [255, 165, 0, 160],    // Orange
       [255, 69, 0, 200],     // Red-Orange
       [255, 0, 0, 240],      // Red (highly unstable)
     ],
   });
   ```

3. **Auto-refresh**: poll `/api/instability` every 60 seconds. On score change, smoothly transition the heatmap intensity.

4. **Performance**: disable on low-GPU devices (detected via `navigator.hardwareConcurrency < 4`). Provide a toggle in the layer controls.

---

← Back to [prd.md](./prd.md)
