# Data Ingestion

> Covers: F08 India Feed Registry · F09 RSS Ingestion Worker · F10 External API Workers · F37 Data Freshness Monitor

---

## F08 — India Feed Registry

### What Is Being Built

A centralized configuration registry that defines every data source BIE monitors — RSS feeds, APIs, scrapers, and manual sources. Each entry specifies the source URL, type, polling frequency, geographic focus, expected data format, and reliability score.

### Why It Is Needed

BIE's intelligence value is directly proportional to the breadth and quality of its data sources. India's security information landscape is fragmented across:

- **Wire services**: PTI (Press Trust of India), ANI, IANS
- **Government sources**: PIB (Press Information Bureau), MEA (Ministry of External Affairs)
- **Defence media**: Indian Defence Research Wing, The Print Defence, NDTV Defence
- **International OSINT**: Reuters Asia, BBC South Asia, SCMP (South China Morning Post)
- **Maritime**: AIS vessel tracking feeds, Indian Navy press releases
- **Regional**: Hindi and Urdu language news sources for internal security

Without a registry:
- Engineers hardcode source URLs in scattered config files
- Adding a new source requires code changes instead of config changes
- There's no visibility into which sources are active, stale, or failing
- Geographic coverage gaps are invisible

### What Use It Provides

- **Single config file** — every data source in one place: `backend/config/feeds.yaml`
- **Hot-reloadable** — add a new source without restarting workers
- **Coverage visibility** — know at a glance which regions and topics are covered
- **Source scoring** — unreliable sources are deprioritized; high-value sources are polled more frequently

### How It Is Built

1. **Feed registry file** — `backend/config/feeds.yaml`:
   ```yaml
   feeds:
     - id: pti_national
       name: "PTI National"
       type: rss
       url: "https://www.ptinews.com/feed/"
       poll_interval_seconds: 300
       category: [MILITARY, DIPLOMATIC, INTERNAL]
       geo_focus: [INDIA_NATIONAL]
       language: en
       reliability: 0.95

     - id: mea_press
       name: "MEA Press Releases"
       type: api
       url: "https://www.mea.gov.in/api/press-releases"
       poll_interval_seconds: 900
       category: [DIPLOMATIC]
       geo_focus: [INDIA_NATIONAL, INTERNATIONAL]
       language: en
       reliability: 0.99

     - id: ais_indian_ocean
       name: "AIS Vessel Tracker — IOR"
       type: api
       url: "https://api.marinetraffic.com/..."
       poll_interval_seconds: 600
       category: [MARITIME]
       geo_focus: [IOR]
       language: en
       reliability: 0.85
       api_key_env: AIS_API_KEY

     - id: hindi_news_feed
       name: "Dainik Jagran Defence"
       type: rss
       url: "https://www.jagran.com/rss/defence.xml"
       poll_interval_seconds: 600
       category: [MILITARY, INTERNAL]
       geo_focus: [INDIA_NATIONAL]
       language: hi
       reliability: 0.80
   ```

2. **Registry loader** — Python module that parses the YAML, validates entries, and provides a typed interface:
   ```python
   class FeedRegistry:
       def __init__(self, config_path="config/feeds.yaml"):
           self.feeds = self._load(config_path)

       def get_rss_feeds(self) -> list[FeedConfig]:
           return [f for f in self.feeds if f.type == "rss"]

       def get_api_feeds(self) -> list[FeedConfig]:
           return [f for f in self.feeds if f.type == "api"]

       def get_by_region(self, region: str) -> list[FeedConfig]:
           return [f for f in self.feeds if region in f.geo_focus]
   ```

3. **Admin endpoint**: `GET /api/feeds/registry` — returns the full registry with last-fetch timestamps and error counts per source.

---

## F09 — RSS Ingestion Worker

### What Is Being Built

A background worker process that continuously polls all RSS feeds from the feed registry, fetches new items, deduplicates them, extracts basic metadata, and publishes them to a Redis Stream for downstream processing by the NLP pipeline.

### Why It Is Needed

RSS feeds are BIE's primary news ingestion channel. Indian wire services (PTI, ANI) publish breaking defence and diplomatic news via RSS before it appears on websites. The CNN effect in modern geopolitics means events unfold in real-time — BIE must capture them within minutes, not hours.

Without an RSS ingestion worker:
- Events are only available when someone manually checks news sites
- There's no pipeline to feed the NLP classifier, knowledge graph, or instability index
- The news feed panel shows nothing

The critical path `F08 → F09 → F11 → F12/F13` means the RSS worker directly feeds the frontend's core data displays.

### What Use It Provides

- **Continuous ingestion** — polls every registered RSS feed at its configured interval
- **Deduplication** — fingerprints each item (hash of title + source + date); skips duplicates
- **Metadata extraction** — pulls title, summary, publication date, author, source link
- **Redis Stream publishing** — downstream consumers (NLP worker) subscribe to `feeds:raw` stream
- **Error resilience** — a failing feed doesn't block other feeds; errors are logged and retried with backoff

### How It Is Built

1. **Worker architecture**:
   ```python
   import asyncio
   import feedparser
   import hashlib
   from datetime import datetime

   class RSSIngestionWorker:
       def __init__(self, registry: FeedRegistry, redis_client):
           self.registry = registry
           self.redis = redis_client
           self.seen_hashes = set()

       async def run(self):
           while True:
               feeds = self.registry.get_rss_feeds()
               tasks = [self.poll_feed(feed) for feed in feeds]
               await asyncio.gather(*tasks, return_exceptions=True)
               await asyncio.sleep(60)  # Main loop interval

       async def poll_feed(self, feed: FeedConfig):
           try:
               parsed = feedparser.parse(feed.url)
               for entry in parsed.entries:
                   fingerprint = hashlib.md5(
                       f"{entry.title}{feed.id}{entry.published}".encode()
                   ).hexdigest()
                   if fingerprint in self.seen_hashes:
                       continue
                   self.seen_hashes.add(fingerprint)

                   item = {
                       "id": fingerprint,
                       "source_id": feed.id,
                       "source_name": feed.name,
                       "title": entry.title,
                       "summary": entry.get("summary", ""),
                       "url": entry.link,
                       "published_at": entry.published,
                       "ingested_at": datetime.utcnow().isoformat(),
                       "language": feed.language,
                       "category_hints": feed.category,
                   }
                   await self.redis.xadd("feeds:raw", item)
           except Exception as e:
               logging.error(f"Error polling {feed.id}: {e}")
   ```

2. **Dedup store**: `seen_hashes` is backed by a Redis SET (`bie:seen_hashes`) with a 48-hour TTL to survive worker restarts.

3. **Backpressure**: if the `feeds:raw` stream exceeds 10,000 unprocessed items, the worker slows polling intervals by 2×.

4. **Metrics**: track per-source ingestion counts, error rates, and latency. Expose via `GET /api/feeds/metrics`.

---

## F10 — External API Workers

### What Is Being Built

Specialized worker processes that poll non-RSS data sources via their HTTP APIs — vessel tracking (AIS), weather data, government press release APIs, and any other structured data feed. Each worker handles the unique authentication, pagination, rate limiting, and data transformation required by its source.

### Why It Is Needed

Not all intelligence data comes via RSS. Critical BIE data sources use proprietary APIs:

- **AIS (Automatic Identification System)** — vessel positions in the Indian Ocean. Essential for the IOR Theater Dashboard (F34) and detecting Chinese naval movements.
- **Government APIs** — MEA press releases, MoD notifications, PIB feeds may have structured JSON endpoints.
- **Weather/environmental data** — natural disaster events that affect regional instability.
- **Social media signals** — structured engagement metrics pointing to emerging crises.

Each API has unique requirements: API keys, rate limits, pagination schemes, and response formats. A generic RSS worker can't handle these.

### What Use It Provides

- **Structured data ingestion** — AIS data gives precise vessel coordinates, not just text mentions
- **Rich metadata** — API responses include structured fields (vessel type, speed, heading) that free-text RSS items lack
- **Rate-limit compliance** — workers respect API quotas, avoiding bans during the 3-week build
- **Unified output** — despite diverse input formats, all workers output to the same `feeds:raw` Redis Stream schema

### How It Is Built

1. **Worker base class**:
   ```python
   class ExternalAPIWorker:
       def __init__(self, feed_config: FeedConfig, redis_client):
           self.config = feed_config
           self.redis = redis_client
           self.rate_limiter = RateLimiter(max_calls=self.config.rate_limit)

       async def run(self):
           while True:
               await self.rate_limiter.wait()
               try:
                   items = await self.fetch()
                   for item in items:
                       normalized = self.normalize(item)
                       await self.redis.xadd("feeds:raw", normalized)
               except Exception as e:
                   logging.error(f"Worker {self.config.id} error: {e}")
               await asyncio.sleep(self.config.poll_interval_seconds)

       async def fetch(self) -> list[dict]:
           raise NotImplementedError

       def normalize(self, raw: dict) -> dict:
           raise NotImplementedError
   ```

2. **AIS Worker** — extends base class:
   ```python
   class AISWorker(ExternalAPIWorker):
       async def fetch(self):
           headers = {"Authorization": f"Bearer {os.getenv('AIS_API_KEY')}"}
           resp = await httpx.get(self.config.url, headers=headers,
               params={"area": "INDIAN_OCEAN", "ship_type": "MILITARY,CARGO"})
           return resp.json()["vessels"]

       def normalize(self, vessel):
           return {
               "id": f"ais_{vessel['mmsi']}_{vessel['timestamp']}",
               "source_id": self.config.id,
               "source_name": "AIS Vessel Tracker",
               "title": f"Vessel {vessel['name']} detected at {vessel['lat']},{vessel['lng']}",
               "summary": f"Type: {vessel['type']}, Speed: {vessel['speed']}kn, Heading: {vessel['heading']}°",
               "lat": vessel["lat"],
               "lng": vessel["lng"],
               "category_hints": ["MARITIME"],
               "ingested_at": datetime.utcnow().isoformat(),
           }
   ```

3. **Worker registry**: a factory that instantiates the correct worker class based on `feed.type`:
   ```python
   WORKER_CLASSES = {
       "ais": AISWorker,
       "government_api": GovernmentAPIWorker,
       "weather": WeatherAPIWorker,
   }
   ```

4. **All workers run concurrently** via `asyncio.gather()` in a single worker process, or distributed across multiple Railway containers for scale.

---

## F37 — Data Freshness Monitor

### What Is Being Built

A monitoring system that tracks how fresh or stale each data source is, displays this information on the frontend, and triggers alerts when a source goes silent. It provides both a backend health-check layer and a frontend UI indicator.

### Why It Is Needed

During a live demo, having stale data is worse than having no data — it looks like the system is working when it's actually showing yesterday's news. The team needs to know:

- Which feeds are actively producing new items?
- Which feeds have gone silent (RSS endpoint changed, API key expired)?
- Is the overall pipeline healthy or is the map showing 6-hour-old events?

### What Use It Provides

- **Per-source freshness** — "PTI: 2 min ago", "AIS: 15 min ago", "MEA: 3 hours ago (⚠️ stale)"
- **Pipeline health** — overall system freshness score: green (all sources < configured threshold), yellow (some stale), red (majority stale)
- **Alert trigger** — if a critical source (PTI, ANI) goes stale for > 30 minutes, log a warning
- **Demo confidence** — before the demo, check the freshness dashboard to verify all feeds are live

### How It Is Built

1. **Backend tracker** — records last-seen timestamp per source:
   ```python
   class FreshnessTracker:
       async def record_ingestion(self, source_id: str):
           await redis.hset("bie:freshness", source_id, datetime.utcnow().isoformat())

       async def get_freshness(self) -> dict:
           raw = await redis.hgetall("bie:freshness")
           result = {}
           for source_id, last_seen in raw.items():
               age = datetime.utcnow() - datetime.fromisoformat(last_seen)
               result[source_id] = {
                   "last_seen": last_seen,
                   "age_seconds": age.total_seconds(),
                   "status": "fresh" if age.total_seconds() < 1800 else "stale"
               }
           return result
   ```

2. **API endpoint**: `GET /api/feeds/freshness` — returns per-source freshness data.

3. **Frontend indicator**: a small colored dot next to each feed source in the News Feed Panel, plus an overall system health indicator in the header.

4. **Alerting**: if any feed with `reliability > 0.9` is stale for > 30 minutes, the backend logs a WARNING-level message and (optionally) sends a notification via webhook.

---

← Back to [prd.md](./prd.md)
