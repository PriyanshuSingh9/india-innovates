# Analytics Engine

> Covers: F14 India Hotspot Database · F25 India Instability Index · F26 Anomaly Detection Engine · F27 Signal Convergence Detector · F33 Infrastructure Proximity

---

## F14 — India Hotspot Database

### What Is Being Built

A curated database of India's geopolitically significant locations with precise coordinates, threat baselines, and metadata. This serves as the geocoding backbone — when the NLP pipeline extracts a location name like "Pangong Tso" or "Galwan Valley", this database maps it to exact coordinates.

### Why It Is Needed

Generic geocoding APIs (Google, Nominatim) return poor results for Indian border landmarks — "Aksai Chin" might resolve to the center of the disputed region rather than the specific flashpoint. BIE needs a purpose-built location database with:
- Precise coordinates for 200+ strategic locations
- Border landmarks along LAC, LOC, IOR
- Military installations, airfields, ports
- Internal hotspots (Naxal corridors, communal tension zones)

### What Use It Provides

- **Accurate geocoding** for India-specific locations
- **Hotspot metadata** — baseline threat level, region classification, category
- **NLP support** — location aliases (Pangong Tso = Pangong Lake = बांगोंग त्सो)
- **Infrastructure lookup** — find military bases within X km of an event

### How It Is Built

1. **Data file** — `data/hotspots/india_hotspots.json`:
   ```json
   [
     {
       "id": "pangong_tso",
       "name": "Pangong Tso",
       "aliases": ["Pangong Lake", "बांगोंग त्सो"],
       "lat": 33.7578, "lng": 78.6611,
       "region": "LAC_WEST",
       "type": "BORDER_LANDMARK",
       "baseline_threat": "HIGH",
       "description": "135km lake, 60% in China. Site of 2020 India-China standoff."
     }
   ]
   ```
2. **Target**: 200+ entries covering LAC, LOC, IOR, Northeast, Naxal corridors.
3. **Lookup API**: `GET /api/hotspots?q={name}` with fuzzy matching and alias support.
4. **NLP integration**: the NLP pipeline's `_geocode()` method queries this database first, falling back to external geocoding only for unknown locations.

---

## F25 — India Instability Index (III)

### What Is Being Built

An algorithmic scoring system that calculates a composite instability score (0.0—1.0) for each monitored region of India. The score is derived from: event frequency, event severity, threat diversity, temporal acceleration, and anomaly signals.

### Why It Is Needed

Individual events tell you *what happened*. The III tells you *how unstable a region is becoming*. Three LOW-threat events in Kashmir in one day might seem routine, but if the baseline is zero events per day, the III flags this as a significant uptick.

The III drives the instability heatmap (F30) and the strategic posture panel (F32). Critical path: `F19 → F25 → F30`.

### What Use It Provides

- **Per-region instability score** — 0.0 (calm) to 1.0 (crisis)
- **Trend direction** — RISING / STABLE / FALLING
- **Contributing factors** — which events are driving the score
- **Heatmap data** — direct feed to the heatmap layer
- **Alert threshold** — score > 0.7 triggers a strategic alert

### How It Is Built

1. **Score formula** (per region, computed every 5 minutes):
   ```
   III = w1 * event_frequency_ratio
       + w2 * avg_threat_severity
       + w3 * category_diversity
       + w4 * temporal_acceleration
       + w5 * anomaly_signals
   ```

   | Component | Weight | Description |
   |---|---|---|
   | Event frequency ratio | 0.25 | Current events / baseline events (7-day rolling) |
   | Avg threat severity | 0.30 | CRITICAL=1.0, HIGH=0.75, MEDIUM=0.5, LOW=0.25 |
   | Category diversity | 0.15 | Number of distinct categories active (normalized) |
   | Temporal acceleration | 0.15 | Rate of change in event frequency |
   | Anomaly signals | 0.15 | Active anomaly count from F26 |

2. **Regions**: LAC_WEST, LAC_EAST, LOC_KASHMIR, SIACHEN, NORTHEAST, IOR, INTERNAL_CENTRAL, INTERNAL_SOUTH.

3. **API endpoint**: `GET /api/instability` returns scores for all regions.

4. **Fallback**: hardcoded baseline scores as per risk register if the pipeline isn't producing enough data.

---

## F26 — Anomaly Detection Engine

### What Is Being Built

A statistical anomaly detection system that identifies unusual patterns in event data — sudden spikes in frequency, unexpected event types in a region, or unusual entity appearances. Anomalies feed into the III score and trigger alerts.

### Why It Is Needed

Intelligence analysis is about finding the unexpected. A single HIGH-threat event might not be anomalous (it happens regularly). But a LOW-threat maritime event in a region that normally has no maritime activity — that's an anomaly worth investigating.

### What Use It Provides

- **Spike detection** — flag when event count in a region exceeds 2σ above the rolling average
- **Category anomalies** — flag events in unexpected categories for a region
- **Entity anomalies** — flag when a new entity (never seen before) appears in a high-threat context
- **Alert generation** — anomalies generate alerts consumed by the posture panel (F32)

### How It Is Built

1. **Statistical baseline**: maintain a 7-day rolling average and standard deviation for event counts per region per category.

2. **Anomaly conditions**:
   ```python
   def detect_anomalies(region: str, current_window: list[Event]) -> list[Anomaly]:
       anomalies = []
       baseline = get_baseline(region)  # 7-day rolling stats

       # Frequency spike
       if len(current_window) > baseline.mean + 2 * baseline.std:
           anomalies.append(Anomaly(type="FREQUENCY_SPIKE", region=region,
               score=(len(current_window) - baseline.mean) / baseline.std))

       # Category anomaly
       for event in current_window:
           if event.category not in baseline.common_categories:
               anomalies.append(Anomaly(type="CATEGORY_ANOMALY", event=event))

       return anomalies
   ```

3. **API**: `GET /api/anomalies/active` returns current anomalies sorted by severity.
4. **Storage**: anomalies stored in Redis with a 24h TTL. Historical anomalies archived in Neo4j.

---

## F27 — Signal Convergence Detector

### What Is Being Built

A system that detects when multiple independent signals converge on the same geographic area within a short time window — indicating a developing situation. Convergence is the most actionable intelligence signal BIE produces.

### Why It Is Needed

Individual events are noise. Convergence is signal. When a region simultaneously shows:
- Military events (troop movements)
- Diplomatic events (ambassador recalled)
- Maritime events (naval deployment nearby)
- Anomaly spike in frequency

...that's convergence — multiple independent indicators pointing to the same place. The demo script shows: *"Convergence alert fires near Aksai Chin. Posture panel escalates to HIGH."*

### What Use It Provides

- **Convergence alerts** — multi-signal events targeting the same region
- **Contributing signals** — which event types are converging
- **Severity amplification** — convergence automatically boosts III score
- **Demo moment** — the most dramatic visual: alert fires, posture escalates, heatmap glows

### How It Is Built

1. **Convergence detection** (runs every 5 minutes):
   ```python
   def detect_convergence(region: str, window_hours: int = 6) -> Optional[Convergence]:
       recent_events = get_events(region=region, since=hours_ago(window_hours))
       categories = set(e.category for e in recent_events)
       anomalies = get_active_anomalies(region)

       # Convergence = 3+ distinct categories + at least 1 anomaly
       if len(categories) >= 3 and len(anomalies) >= 1:
           return Convergence(
               region=region,
               categories=list(categories),
               event_count=len(recent_events),
               anomaly_count=len(anomalies),
               severity="HIGH" if len(categories) >= 4 else "MEDIUM",
           )
       return None
   ```

2. **Alert**: convergence detection pushes alerts to `alerts:convergence` Redis Stream, consumed by the posture panel and SSE endpoint.

3. **Demo inject**: `POST /demo/inject` can seed a scripted convergence scenario to ensure it fires during the demo.

---

## F33 — Infrastructure Proximity

### What Is Being Built

A spatial analysis feature that, for any event or location, identifies nearby strategic infrastructure — military bases, airfields, ports, radar stations, missile sites — within a configurable radius. This adds strategic context to every event.

### Why It Is Needed

An event "near Pangong Tso" is one thing. Knowing it's 15km from Daulat Beg Oldi (India's highest airstrip) and 40km from a Chinese military outpost transforms it into actionable intelligence. Infrastructure proximity turns geographic coordinates into strategic context.

The demo: *"Click the event marker. Show EventDetailPanel with extracted entities and proximity infrastructure."*

### What Use It Provides

- **Proximity list** — for any event, show which infrastructure is within 50km, 100km, 200km
- **Distance + bearing** — "DBO Airstrip: 15km NNE"
- **Risk assessment** — infrastructure near hostile events gets flagged
- **Map visualization** — draw radius rings and infrastructure markers around a selected event

### How It Is Built

1. **Infrastructure data**: sourced from India Hotspot Database (F14), filtered to type = INFRASTRUCTURE.
2. **Haversine distance calculation**:
   ```python
   from math import radians, sin, cos, sqrt, atan2

   def haversine(lat1, lng1, lat2, lng2) -> float:
       R = 6371  # Earth radius in km
       dlat = radians(lat2 - lat1)
       dlng = radians(lng2 - lng1)
       a = sin(dlat/2)**2 + cos(radians(lat1)) * cos(radians(lat2)) * sin(dlng/2)**2
       return R * 2 * atan2(sqrt(a), sqrt(1-a))
   ```
3. **API**: `GET /api/proximity?lat={}&lng={}&radius_km=50` returns nearby infrastructure sorted by distance.
4. **Frontend**: EventDetailPanel includes a "Nearby Infrastructure" section with distance badges.

---

← Back to [prd.md](./prd.md)
