# Intelligence Products

> Covers: F31 Strategic Brief Generator

---

## F31 — Strategic Brief Generator

### What Is Being Built

An automated report generator that produces a structured **India Strategic Intelligence Brief** every 6 hours. The brief synthesizes all ingested events, instability scores, anomalies, and convergence alerts into a readable, actionable document — the kind of intelligence product a decision-maker would receive on their desk.

### Why It Is Needed

Dashboards are great for real-time monitoring, but decision-makers need *summaries*. A general doesn't stare at a live map all day — they read a brief over morning tea. BIE's brief generator transforms the live data stream into a consumable intelligence product.

The demo script features this prominently: *"Click Generate Brief. Show India Strategic Intelligence Brief generating in real time. Mention it auto-generates every 6 hours."* This is what makes BIE an intelligence **engine**, not just a dashboard.

### What Use It Provides

- **Auto-generated briefs** every 6 hours covering the last period
- **On-demand generation** — analysts can click "Generate Brief" anytime
- **Structured sections** — Executive Summary, Regional Analysis, Key Events, Threat Assessment, Recommendations
- **LLM-powered writing** — Claude generates natural-language analysis from structured data
- **Instability integration** — brief includes III scores and trends per region
- **Streaming generation** — brief appears word-by-word in the UI for real-time feel
- **PDF export** — option to export the brief as a formatted PDF

### How It Is Built

1. **Brief data assembly** — gather inputs for Claude prompt:
   ```python
   async def assemble_brief_context(period_hours: int = 6) -> dict:
       events = await get_recent_events(hours=period_hours)
       instability = await get_instability_scores()
       anomalies = await get_recent_anomalies(hours=period_hours)
       convergences = await get_active_convergences()

       return {
           "period_start": datetime.utcnow() - timedelta(hours=period_hours),
           "period_end": datetime.utcnow(),
           "total_events": len(events),
           "events_by_category": group_by(events, "category"),
           "events_by_region": group_by(events, "region"),
           "high_threat_events": [e for e in events if e.threat_level in ("HIGH", "CRITICAL")],
           "instability_scores": instability,
           "anomalies": anomalies,
           "convergences": convergences,
       }
   ```

2. **Claude prompt**:
   ```
   You are a senior intelligence analyst writing the India Strategic Intelligence
   Brief for the period {period_start} to {period_end}.

   Structure your brief in these sections:
   1. EXECUTIVE SUMMARY (3-4 sentences, most critical developments)
   2. REGIONAL ANALYSIS (per region: LAC, LOC, IOR, Northeast, Internal)
   3. KEY EVENTS (top 5 events with analysis)
   4. THREAT ASSESSMENT (overall posture, instability trends)
   5. WATCH ITEMS (emerging patterns, things to monitor)

   DATA:
   {brief_context}

   Write in a professional, concise intelligence style. Cite events by ID.
   ```

3. **Streaming endpoint**: `POST /api/brief/generate` returns a streaming response.

4. **Caching**: generated briefs are cached for 6 hours (matching the generation interval). On-demand generation always produces a fresh brief.

5. **Scheduled generation**: a cron-style task triggers brief generation every 6 hours:
   ```python
   # Using APScheduler or similar
   scheduler.add_job(generate_brief, "interval", hours=6)
   ```

6. **API endpoints**:
   | Method | Path | Description |
   |---|---|---|
   | `POST` | `/api/brief/generate` | Generate a new brief (streaming) |
   | `GET` | `/api/brief/latest` | Get the most recent cached brief |
   | `GET` | `/api/brief/{id}` | Get a specific historical brief |
   | `GET` | `/api/brief/history` | List all generated briefs |

---

← Back to [prd.md](./prd.md)
