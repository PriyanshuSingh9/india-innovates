# UI Panels

> Covers: F12 News Feed Panel · F28 NL Query Interface · F29 Knowledge Graph Panel · F32 Strategic Posture Panel · F34 IOR Theater Dashboard

---

## F12 — News Feed Panel

### What Is Being Built

A real-time scrolling news feed panel on the left side of the BIE interface. It displays ingested and NLP-classified intelligence items in reverse chronological order, with filtering by category, threat level, and source. Clicking an item flies the map to the event's location and opens its detail panel.

### Why It Is Needed

The map shows *where* events are; the news feed shows *what* they are. Together they create the core BIE experience: a spatial + textual intelligence view. Analysts need to scan recent events quickly, spot patterns by category, and drill into specific items.

The demo script requires: *"A live news item arrives in the feed (PTI wire). Click it. Map flies to the event location."* This panel is where that interaction starts.

### What Use It Provides

- **Real-time feed** — new items appear at the top as they are ingested (via SSE)
- **Category badges** — MILITARY (red), DIPLOMATIC (blue), ECONOMIC (green), INTERNAL (orange), MARITIME (teal)
- **Threat-level indicators** — colored dot next to each item (green/yellow/red/pulsing red)
- **Source attribution** — "PTI", "ANI", "MEA", "AIS" labels
- **Click-to-fly** — clicking an item triggers `map.flyTo()` to the event's coordinates
- **Filter bar** — filter by category, threat level, time window, source
- **Infinite scroll** — paginated loading via cursor-based API

### How It Is Built

1. **Component structure**:
   ```
   components/
   ├── NewsFeedPanel/
   │   ├── NewsFeedPanel.js     — Container with filter bar + list
   │   ├── FeedItem.js          — Individual news item card
   │   ├── FeedFilter.js        — Filter controls (category, threat, source)
   │   └── feedPanel.css        — Panel-specific styles
   ```

2. **Data flow**:
   - Initial load: `GET /api/feeds/items?limit=50`
   - Real-time updates: subscribe to `EventSource("/api/events/stream")`
   - New items are prepended to the list with a slide-in animation

3. **FeedItem component**:
   ```jsx
   function FeedItem({ item, onSelect }) {
     return (
       <div className="feed-item" onClick={() => onSelect(item)}>
         <div className="feed-item-header">
           <span className={`badge badge-${item.category.toLowerCase()}`}>
             {item.category}
           </span>
           <span className={`threat-dot threat-${item.threat_level.toLowerCase()}`} />
           <span className="feed-time">{timeAgo(item.timestamp)}</span>
         </div>
         <h4 className="feed-title">{item.title}</h4>
         <p className="feed-summary">{item.summary}</p>
         <span className="feed-source">{item.source}</span>
       </div>
     );
   }
   ```

4. **Panel layout**: fixed-width left panel (380px), full height, scrollable. Dark background matching the map theme. Semi-transparent with glassmorphism effect.

---

## F28 — NL Query Interface

### What Is Being Built

A natural-language query bar where analysts type intelligence questions in plain English. The query is processed by the GraphRAG engine (F23), which searches the Neo4j knowledge graph and Qdrant vector store, then generates a streaming LLM response grounded in real data, with citations.

### Why It Is Needed

Traditional intelligence systems require analysts to write SQL queries or navigate complex filter UIs. BIE lets them ask questions like a conversation:

- *"What is China doing near the LAC this week and how does it compare to last month?"*
- *"Show me all maritime incidents in the Strait of Malacca involving Chinese vessels"*
- *"What are the top 3 threats to India's northeastern border?"*

The demo script features this moment prominently: the presenter types a question and gets a streaming, cited answer. This is BIE's most impressive capability.

### What Use It Provides

- **Natural language input** — type questions in plain English, no query language needed
- **Streaming response** — answer appears word-by-word, like ChatGPT, maintaining engagement
- **Citations** — every factual claim links back to its source event or graph entity
- **Confidence score** — shows how confident the system is in the answer (< 0.6 triggers a disclaimer)
- **Graph context** — optionally shows the subgraph traversed to generate the answer
- **Query history** — recent queries are saved for quick re-access

### How It Is Built

1. **Query bar UI** — centered at the top of the map or in a dedicated panel:
   ```jsx
   function QueryBar({ onSubmit }) {
     const [query, setQuery] = useState("");
     return (
       <form onSubmit={(e) => { e.preventDefault(); onSubmit(query); }}>
         <input
           type="text"
           value={query}
           onChange={(e) => setQuery(e.target.value)}
           placeholder="Ask BIE anything about India's security environment..."
           className="query-input"
         />
         <button type="submit" className="query-submit">Ask</button>
       </form>
     );
   }
   ```

2. **Streaming response** — use `fetch` with `ReadableStream`:
   ```javascript
   async function streamQuery(query, onChunk) {
     const response = await fetch("/api/query", {
       method: "POST",
       body: JSON.stringify({ query }),
       headers: { "Content-Type": "application/json" },
     });
     const reader = response.body.getReader();
     const decoder = new TextDecoder();
     while (true) {
       const { done, value } = await reader.read();
       if (done) break;
       onChunk(decoder.decode(value));
     }
   }
   ```

3. **Response panel** — displays below the query bar:
   - Streaming text with markdown rendering
   - Citation chips: clickable, fly-to the cited event on the map
   - Confidence bar: colored green (>0.8), yellow (0.6-0.8), red (<0.6 with ⚠️ disclaimer)
   - "Show graph context" toggle: reveals the Neo4j subgraph used

4. **Fallback**: if GraphRAG fails (Neo4j down, Claude timeout), fall back to Qdrant `/similar` endpoint — returns semantically similar events without LLM synthesis, with a message: "Full analysis unavailable. Showing related events."

---

## F29 — Knowledge Graph Panel

### What Is Being Built

A visual panel that renders a subset of the Neo4j knowledge graph as an interactive node-link diagram. Users can explore entities (people, organizations, locations, events), see their relationships, and click nodes to drill into detail.

### Why It Is Needed

The knowledge graph is BIE's "brain" — it stores relationships between entities extracted from thousands of news items. But a graph database is useless if analysts can't see and explore it. The KG panel makes the invisible visible:

- Who is connected to whom?
- Which organizations are active in which regions?
- What events link two entities together?

### What Use It Provides

- **Entity visualization** — nodes colored by type (person=blue, org=red, location=green, event=yellow)
- **Relationship edges** — labeled connections (MENTIONED_IN, OPERATES_IN, RELATED_TO)
- **Interactive exploration** — click a node to expand its connections; double-click to center the graph on it
- **Search** — type an entity name to find and highlight it in the graph
- **Context-aware** — when viewing an event, the KG panel shows the subgraph related to that event

### How It Is Built

1. **Visualization library**: use `react-force-graph-2d` or `vis-network` for interactive graph rendering.

2. **Data source**: `GET /api/graph/search?entity={name}` returns a subgraph:
   ```json
   {
     "nodes": [
       { "id": "org_pla", "label": "PLA", "type": "organization" },
       { "id": "loc_galwan", "label": "Galwan Valley", "type": "location" },
       { "id": "event_123", "label": "PLA patrol detected", "type": "event" }
     ],
     "edges": [
       { "source": "org_pla", "target": "loc_galwan", "label": "OPERATES_IN" },
       { "source": "event_123", "target": "org_pla", "label": "INVOLVES" }
     ]
   }
   ```

3. **Graph component**:
   ```jsx
   import ForceGraph2D from "react-force-graph-2d";

   function KnowledgeGraphPanel({ graphData, onNodeClick }) {
     return (
       <ForceGraph2D
         graphData={graphData}
         nodeLabel="label"
         nodeColor={(node) => NODE_COLORS[node.type]}
         linkLabel="label"
         onNodeClick={onNodeClick}
         width={400}
         height={500}
         backgroundColor="#1a1a2e"
       />
     );
   }
   ```

4. **Panel placement**: right-side collapsible panel, below the query interface. Opens when a query returns graph context, or when user clicks "Show Knowledge Graph".

---

## F32 — Strategic Posture Panel

### What Is Being Built

A dashboard panel showing India's current strategic posture across key dimensions: overall threat level, per-region instability scores, active anomalies, and signal convergence alerts. It provides a summary view of the India Instability Index (III) and the anomaly/convergence engines.

### Why It Is Needed

The map shows *geography*, the feed shows *events*, but the posture panel shows *assessment*. It answers: "How worried should we be right now?" This is what makes BIE an intelligence *engine* rather than just a news aggregator.

The demo script requires: *"Convergence alert fires near Aksai Chin. Posture panel escalates to HIGH."* The posture panel is where analysts see the systemic threat level change in real-time.

### What Use It Provides

- **Overall posture indicator** — NORMAL / ELEVATED / HIGH / CRITICAL with color coding
- **Per-region scores** — bar chart showing III scores for each region, sorted by severity
- **Active alerts** — list of current anomaly and convergence alerts with timestamps
- **Trend indicators** — ↑ RISING / → STABLE / ↓ FALLING per region
- **Contributing factors** — for each high-scoring region, list the events driving the score
- **Historical mini-chart** — sparkline showing posture over the last 24 hours

### How It Is Built

1. **Data source**:
   - `GET /api/instability` — per-region III scores
   - `GET /api/instability/posture` — overall posture level
   - `GET /api/anomalies/active` — current anomaly alerts
   - `GET /api/convergence/active` — current convergence alerts

2. **Posture level logic**:
   ```
   NORMAL:   all regions < 0.4
   ELEVATED: any region 0.4–0.6
   HIGH:     any region 0.6–0.8
   CRITICAL: any region > 0.8 OR convergence alert active
   ```

3. **UI layout**: bottom-right panel or full-width bottom bar. Contains:
   - Large posture badge (color-coded)
   - Region score bars (horizontal, sorted descending)
   - Alert list (scrollable, most recent first)
   - Sparkline (last 24h posture history)

4. **Real-time updates**: poll every 30 seconds. On posture level change, trigger a subtle pulse animation on the badge and (if escalating) a brief sound indicator.

---

## F34 — IOR Theater Dashboard

### What Is Being Built

A specialized sub-dashboard focused entirely on the Indian Ocean Region (IOR). It combines vessel tracking data (from AIS feeds), maritime event markers, chokepoint monitoring, and Chinese naval presence indicators into a dedicated maritime intelligence view.

### Why It Is Needed

India's maritime domain awareness is a critical national security priority. The IOR spans from the Strait of Hormuz to the Strait of Malacca — controlling supply routes for 80% of global oil trade. China's expanding naval presence (String of Pearls strategy) makes IOR monitoring essential.

The demo closes with: *"Zoom out to IOR. Show Chinese vessel tracker. 'Palantir charges sovereign governments $50M for this. We built it for India in 3 weeks.'"* The IOR dashboard is this closing moment.

### What Use It Provides

- **Vessel positions** — real-time dots showing ship locations from AIS data
- **Vessel filtering** — filter by type (military, cargo, fishing), flag state (China, India, Pakistan)
- **Chokepoint monitoring** — traffic counts at Malacca, Hormuz, Bab el-Mandeb
- **Chinese naval overlay** — highlighted markers for Chinese naval vessels and bases (Djibouti, Gwadar, Hambantota)
- **Maritime events** — piracy, illegal fishing, unusual military movements
- **Region preset** — one click to switch from the India land view to the IOR maritime view

### How It Is Built

1. **Data sources**:
   - AIS API worker (F10) for vessel positions
   - Maritime event markers from RSS feeds categorized as MARITIME
   - Static data for Chinese base locations (Djibouti, Gwadar, Hambantota)

2. **Vessel layer** (Deck.gl IconLayer):
   ```javascript
   const vesselLayer = new IconLayer({
     id: "vessels",
     data: vessels,
     getPosition: (d) => [d.lng, d.lat],
     getIcon: (d) => vesselIcon(d.type, d.flag),
     getSize: 24,
     pickable: true,
     onClick: ({ object }) => setSelectedVessel(object),
   });
   ```

3. **Chinese naval overlay**: static dataset of known base locations and patrol routes, shown as distinct red markers and route arcs.

4. **Panel UI**: when IOR preset is active, a side panel shows:
   - Vessel count by flag state
   - Chokepoint traffic summary
   - Recent maritime events
   - Threat assessment for the IOR region

---

← Back to [prd.md](./prd.md)
