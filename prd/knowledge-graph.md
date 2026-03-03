# Knowledge Graph & Vector Search

> Covers: F21 Neo4j Schema & Seed Data · F22 Automated Graph Writer · F23 GraphRAG Query Engine · F24 Qdrant Vector Store

---

## F21 — Neo4j Schema & Seed Data

### What Is Being Built

The Neo4j graph database schema defining all node types, relationship types, property constraints, and indexes — plus a seed dataset of 1,000 curated facts about India's geopolitical environment to ensure the graph is richly populated before the demo.

### Why It Is Needed

The knowledge graph is BIE's long-term memory. Individual news events are ephemeral; the graph captures the *relationships* between entities across events. "PLA deployed troops near Pangong Tso" and "India moved 3 Bihar Regiment to LAC" are separate events, but the graph connects PLA → Pangong Tso → LAC → 3 Bihar Regiment — revealing the confrontation network.

Without a schema, graph data is unstructured and unqueryable. Without seed data, the graph is empty at demo time — the risk register flags this as **High** risk, mitigated by seeding 1,000 facts manually by Day 8.

### What Use It Provides

- **Structured graph** — typed nodes and relationships with property constraints
- **Queryable** — Cypher queries can traverse relationships efficiently
- **Pre-populated** — 1,000 seed facts mean the demo shows a rich, interconnected graph from the start
- **Extensible** — the automated graph writer (F22) adds to this schema dynamically

### How It Is Built

1. **Node types**:

   | Label | Properties | Description |
   |---|---|---|
   | `Person` | name, role, nationality, aliases | World leaders, military commanders, diplomats |
   | `Organization` | name, type, country, aliases | PLA, Indian Army, DRDO, RAW, ISI |
   | `Location` | name, lat, lng, type, region | Pangong Tso, Galwan Valley, Gwadar Port |
   | `Event` | id, title, summary, timestamp, category, threat_level | Intelligence events |
   | `MilitaryUnit` | name, branch, country | 3 Bihar Regiment, PLA Western Theater |
   | `WeaponSystem` | name, type, country | BrahMos, S-400, J-20 |
   | `Infrastructure` | name, type, lat, lng | Daulat Beg Oldi airstrip, Hambantota Port |

2. **Relationship types**:

   | Type | From → To | Description |
   |---|---|---|
   | `MENTIONED_IN` | Entity → Event | Entity was mentioned in this event |
   | `OPERATES_IN` | Org/Unit → Location | Organization operates in this region |
   | `DEPLOYED_AT` | Unit → Location | Military unit deployed at location |
   | `ALLIES_WITH` | Org → Org | Allied organizations |
   | `RIVALS_WITH` | Org → Org | Rival organizations |
   | `NEAR` | Location → Location | Geographic proximity (< 100km) |
   | `PART_OF` | Location → Location | Hierarchical (city → state → country) |
   | `COMMANDS` | Person → Org/Unit | Person commands organization |

3. **Seed data script** — `backend/scripts/seed_neo4j.py`: insert 1,000 pre-curated facts about India-China-Pakistan geopolitics, military deployments, key personalities, and infrastructure.

4. **Indexes**: create indexes on `Person.name`, `Organization.name`, `Location.name`, `Event.id` for fast lookups.

---

## F22 — Automated Graph Writer

### What Is Being Built

A service that consumes NLP-enriched items from `feeds:processed` and automatically creates/updates nodes and relationships in Neo4j. It maps extracted entities to graph nodes, infers relationships from co-occurrence in events, and links everything to Event nodes.

### Why It Is Needed

Manual graph population doesn't scale. BIE ingests dozens of events per hour. Each event mentions 2–5 entities. The graph writer must automatically:
- Create new entity nodes when first seen
- Link existing entities to new events
- Infer relationships (entities co-mentioned = likely related)
- Deduplicate (merge "PM Modi" with "Narendra Modi")

### What Use It Provides

- **Automatic graph growth** — the knowledge graph grows organically as news arrives
- **Entity deduplication** — fuzzy matching merges variant names into single nodes
- **Relationship inference** — co-occurrence in events creates implicit connections
- **Event linking** — every event connects to the entities it mentions

### How It Is Built

1. **Graph writer service** — `backend/app/services/graph_writer.py`:
   ```python
   class GraphWriter:
       def __init__(self, neo4j_driver):
           self.driver = neo4j_driver

       async def write_event(self, enriched_item: dict):
           async with self.driver.session() as session:
               # Create Event node
               await session.run("""
                   MERGE (e:Event {id: $id})
                   SET e.title = $title, e.summary = $summary,
                       e.category = $category, e.threat_level = $threat_level,
                       e.timestamp = datetime($timestamp)
               """, enriched_item)

               # Create/link entity nodes
               for entity in enriched_item["entities"]:
                   label = ENTITY_TYPE_MAP.get(entity["label"], "Entity")
                   await session.run(f"""
                       MERGE (n:{label} {{name: $name}})
                       MERGE (e:Event {{id: $event_id}})
                       MERGE (n)-[:MENTIONED_IN]->(e)
                   """, {"name": entity["text"], "event_id": enriched_item["id"]})
   ```

2. **Entity deduplication**: use fuzzy string matching (Levenshtein distance < 3) to merge "PM Modi" → "Narendra Modi".

3. **Relationship inference**: if two non-Event entities co-occur in 3+ events, create a `RELATED_TO` relationship.

4. **Runs as**: a consumer on the `feeds:processed` Redis Stream, alongside or within the NLP worker process.

---

## F23 — GraphRAG Query Engine

### What Is Being Built

A Retrieval-Augmented Generation (RAG) engine that combines Neo4j graph traversal with Qdrant vector search and Claude LLM synthesis. When an analyst asks a natural-language question, GraphRAG retrieves relevant graph context and similar documents, then prompts Claude to generate a grounded, cited answer.

### Why It Is Needed

The knowledge graph has data. The vector store has embeddings. But analysts ask questions in English, not Cypher or vector queries. GraphRAG bridges this gap — it's the engine behind the NL Query Interface (F28).

The demo script requires: *"Type a query. Watch streaming answer with citations. GraphRAG working. LLM grounded in real graph data."*

### What Use It Provides

- **Natural language → structured answer** — ask in English, get a cited analysis
- **Graph-grounded** — answers are based on real data in Neo4j, not LLM hallucination
- **Vector-enhanced** — Qdrant finds semantically similar events for broader context
- **Streaming** — answer streams word-by-word for responsive UX
- **Citations** — every factual claim links to its source event
- **Confidence score** — if < 0.6, show disclaimer per risk register

### How It Is Built

1. **Query pipeline**:
   - **Step 1**: Extract entities from the query using spaCy NER
   - **Step 2**: Search Neo4j for subgraphs connected to those entities (2-hop traversal)
   - **Step 3**: Search Qdrant for top-10 semantically similar events
   - **Step 4**: Construct a Claude prompt with graph context + similar events
   - **Step 5**: Stream Claude's response back to the frontend

2. **Claude prompt template**:
   ```
   You are BIE, an intelligence analyst for India. Answer the query using ONLY
   the provided context. Cite sources using [Event:ID] notation.

   GRAPH CONTEXT:
   {neo4j_subgraph}

   SIMILAR EVENTS:
   {qdrant_results}

   QUERY: {user_query}
   ```

3. **Streaming**: use Anthropic's streaming API to yield tokens as they arrive.

4. **Fallback**: if Neo4j is down, use only Qdrant results. If Claude is down, return Qdrant results as a list without LLM synthesis.

---

## F24 — Qdrant Vector Store

### What Is Being Built

A Qdrant vector database that stores embeddings for every processed event. These embeddings enable semantic similarity search — finding events that are conceptually related even if they don't share keywords.

### Why It Is Needed

Keyword search misses conceptual relationships. "Chinese military buildup near Aksai Chin" and "PLA troops mobilized along Western LAC" are about the same thing but share few keywords. Vector embeddings capture semantic meaning, making similarity search work across different phrasings.

Qdrant also serves as a fallback for GraphRAG (F23) when Neo4j is down — the risk register specifies "Qdrant `/similar` as fallback answer."

### What Use It Provides

- **Semantic search** — find conceptually similar events regardless of exact wording
- **GraphRAG context** — provides the "SIMILAR EVENTS" component of RAG prompts
- **Fallback query** — when GraphRAG fails, Qdrant `/similar` returns relevant events directly
- **Fast retrieval** — vector similarity search in < 50ms for 10,000 documents

### How It Is Built

1. **Setup**: Qdrant instance on Railway. Collection `bie_events`.
2. **Embeddings**: use `sentence-transformers/all-MiniLM-L6-v2` (384 dimensions, fast, good quality):
   ```python
   from sentence_transformers import SentenceTransformer
   model = SentenceTransformer("all-MiniLM-L6-v2")

   def embed_event(event: dict) -> list[float]:
       text = f"{event['title']} {event['summary']}"
       return model.encode(text).tolist()
   ```
3. **Indexing**: every processed event is embedded and upserted into Qdrant.
4. **Search endpoint**: `GET /api/similar?q={text}&limit=10` returns the most semantically similar events.
5. **Demo seeding**: pre-embed 10,000 documents so semantic search returns strong results immediately.

---

← Back to [prd.md](./prd.md)
