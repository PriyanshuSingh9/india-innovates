# Data Schemas

> Central schema reference for all data models used across BIE subsystems.

---

## 1. API Response Schemas

### Event

```json
{
  "id": "string — unique event fingerprint (MD5 hash)",
  "title": "string — event headline",
  "summary": "string — event summary/description",
  "source": "string — source feed name (e.g. PTI, ANI)",
  "source_id": "string — feed registry ID",
  "url": "string — original source URL",
  "category": "enum — MILITARY | DIPLOMATIC | ECONOMIC | INTERNAL | MARITIME",
  "threat_level": "enum — LOW | MEDIUM | HIGH | CRITICAL",
  "location": {
    "lat": "float — latitude",
    "lng": "float — longitude",
    "name": "string — resolved location name"
  },
  "entities": [
    {
      "text": "string — entity surface text",
      "label": "string — PERSON | ORG | GPE | LOC | MILITARY_UNIT | WEAPON_SYSTEM | INFRASTRUCTURE | BORDER_LANDMARK | INDIAN_ORG",
      "start": "int — character offset start",
      "end": "int — character offset end"
    }
  ],
  "classification_confidence": "float — 0.0 to 1.0",
  "timestamp": "ISO 8601 datetime — event publication time",
  "ingested_at": "ISO 8601 datetime — BIE ingestion time",
  "processed_at": "ISO 8601 datetime — NLP processing time"
}
```

### FeedItem (raw)

```json
{
  "id": "string — fingerprint",
  "source_id": "string — feed registry ID",
  "source_name": "string — human-readable source name",
  "title": "string",
  "summary": "string",
  "url": "string — original article URL",
  "published_at": "ISO 8601 datetime",
  "ingested_at": "ISO 8601 datetime",
  "language": "string — en | hi | ur",
  "category_hints": ["string — categories from feed registry"]
}
```

### InstabilityScore

```json
{
  "region": "string — LAC_WEST | LAC_EAST | LOC_KASHMIR | SIACHEN | NORTHEAST | IOR | INTERNAL_CENTRAL | INTERNAL_SOUTH",
  "score": "float — 0.0 to 1.0",
  "trend": "enum — RISING | STABLE | FALLING",
  "contributing_factors": ["string — event IDs driving the score"],
  "event_count": "int — events in current window",
  "baseline_event_count": "float — 7-day rolling average",
  "last_updated": "ISO 8601 datetime"
}
```

### QueryResponse

```json
{
  "answer": "string — LLM-generated analysis (streamed)",
  "citations": [
    {
      "event_id": "string",
      "title": "string",
      "relevance": "float"
    }
  ],
  "confidence": "float — 0.0 to 1.0",
  "graph_context": {
    "nodes": ["..."],
    "edges": ["..."]
  },
  "fallback_used": "boolean — true if Qdrant-only fallback was used"
}
```

### StrategicBrief

```json
{
  "id": "string — UUID",
  "generated_at": "ISO 8601 datetime",
  "period_start": "ISO 8601 datetime",
  "period_end": "ISO 8601 datetime",
  "sections": [
    {
      "title": "string — section name",
      "content": "string — markdown content",
      "events_referenced": ["string — event IDs"]
    }
  ],
  "instability_scores": {
    "LAC_WEST": 0.75,
    "LOC_KASHMIR": 0.82
  },
  "overall_posture": "enum — NORMAL | ELEVATED | HIGH | CRITICAL"
}
```

### Anomaly

```json
{
  "id": "string — UUID",
  "type": "enum — FREQUENCY_SPIKE | CATEGORY_ANOMALY | ENTITY_ANOMALY",
  "region": "string",
  "score": "float — deviation from baseline",
  "event_ids": ["string — triggering events"],
  "detected_at": "ISO 8601 datetime",
  "expires_at": "ISO 8601 datetime — 24h TTL"
}
```

### Convergence

```json
{
  "id": "string — UUID",
  "region": "string",
  "categories": ["string — converging categories"],
  "event_count": "int",
  "anomaly_count": "int",
  "severity": "enum — MEDIUM | HIGH",
  "detected_at": "ISO 8601 datetime"
}
```

---

## 2. Neo4j Graph Schema

### Node Labels & Properties

| Label | Properties | Constraints |
|---|---|---|
| `Person` | name (string), role (string), nationality (string), aliases (string[]) | UNIQUE on name |
| `Organization` | name (string), type (string), country (string), aliases (string[]) | UNIQUE on name |
| `Location` | name (string), lat (float), lng (float), type (string), region (string) | UNIQUE on name |
| `Event` | id (string), title (string), summary (string), category (string), threat_level (string), timestamp (datetime) | UNIQUE on id |
| `MilitaryUnit` | name (string), branch (string), country (string) | UNIQUE on name |
| `WeaponSystem` | name (string), type (string), country (string) | UNIQUE on name |
| `Infrastructure` | name (string), type (string), lat (float), lng (float) | UNIQUE on name |

### Relationship Types

| Type | From → To | Properties |
|---|---|---|
| `MENTIONED_IN` | Entity → Event | role (string), confidence (float) |
| `OPERATES_IN` | Org/Unit → Location | since (date) |
| `DEPLOYED_AT` | MilitaryUnit → Location | since (date), status (string) |
| `ALLIES_WITH` | Org → Org | — |
| `RIVALS_WITH` | Org → Org | — |
| `NEAR` | Location → Location | distance_km (float) |
| `PART_OF` | Location → Location | — (hierarchy: city → state → country) |
| `COMMANDS` | Person → Org/Unit | title (string) |
| `RELATED_TO` | Entity → Entity | co_occurrence_count (int) |

### Indexes

- `CREATE INDEX ON :Person(name)`
- `CREATE INDEX ON :Organization(name)`
- `CREATE INDEX ON :Location(name)`
- `CREATE INDEX ON :Event(id)`
- `CREATE INDEX ON :Event(timestamp)`
- `CREATE INDEX ON :Event(category)`

---

## 3. Qdrant Collection Schema

### Collection: `bie_events`

| Field | Type | Description |
|---|---|---|
| **vector** | float[384] | Embedding from `all-MiniLM-L6-v2` |
| **payload.id** | string | Event ID (matches Event.id) |
| **payload.title** | string | Event title |
| **payload.summary** | string | Event summary |
| **payload.category** | string | Event category |
| **payload.threat_level** | string | Threat level |
| **payload.region** | string | Geographic region |
| **payload.timestamp** | string | ISO 8601 timestamp |
| **payload.entities** | string[] | Extracted entity names |

### Indexes

- Payload index on `category` (keyword)
- Payload index on `threat_level` (keyword)
- Payload index on `region` (keyword)
- Payload index on `timestamp` (datetime range)

---

## 4. Redis Data Structures

### Streams

| Stream | Purpose | Producers | Consumers |
|---|---|---|---|
| `feeds:raw` | Raw ingested items | RSS Worker, API Workers | NLP Worker |
| `feeds:processed` | NLP-enriched items | NLP Worker | Graph Writer, Event API |
| `feeds:dead_letter` | Failed processing items | NLP Worker | Manual review |
| `alerts:convergence` | Convergence alerts | Convergence Detector | Posture Panel (SSE) |
| `events:new` | New events for SSE | Event API | Frontend (SSE) |

### Hashes

| Key Pattern | Purpose |
|---|---|
| `bie:freshness` | Per-source last-seen timestamps |
| `bie:instability:{region}` | Current III scores per region |
| `bie:baseline:{region}` | 7-day rolling stats per region |

### Sets

| Key Pattern | Purpose |
|---|---|
| `bie:seen_hashes` | Dedup fingerprints (48h TTL) |

### Strings (cached responses)

| Key Pattern | TTL | Purpose |
|---|---|---|
| `bie:cache:events:{hash}` | 60s | Event API response cache |
| `bie:cache:instability:{hash}` | 60s | Instability API response cache |
| `bie:cache:llm:{hash}` | 24h | Claude API response cache |
| `bie:cache:brief:latest` | 6h | Latest generated brief |

---

## 5. Feed Registry Schema (YAML)

```yaml
feeds:
  - id: string            # Unique feed identifier
    name: string           # Human-readable name
    type: enum             # rss | api | scraper
    url: string            # Feed URL
    poll_interval_seconds: int  # Polling frequency
    category: [string]     # Expected categories
    geo_focus: [string]    # Geographic focus regions
    language: string       # en | hi | ur
    reliability: float     # 0.0 to 1.0
    api_key_env: string    # Optional env var name for API key
    rate_limit: int        # Optional max requests per minute
```

---

## 6. India Hotspot Schema (JSON)

```json
{
  "id": "string — unique identifier (snake_case)",
  "name": "string — primary name",
  "aliases": ["string — alternate names, including Devanagari"],
  "lat": "float — latitude",
  "lng": "float — longitude",
  "region": "string — LAC_WEST | LAC_EAST | LOC_KASHMIR | SIACHEN | NORTHEAST | IOR | INTERNAL_*",
  "type": "string — BORDER_LANDMARK | MILITARY_BASE | AIRFIELD | PORT | CITY | PASS | GLACIER | STRAIT",
  "baseline_threat": "string — LOW | MEDIUM | HIGH",
  "country": "string — IN | CN | PK | LK | MM",
  "description": "string — brief strategic significance"
}
```

---

← Back to [prd.md](./prd.md)
