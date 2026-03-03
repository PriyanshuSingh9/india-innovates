# 🇮🇳 Bharat Intelligence Engine (BIE)

> *India spends billions on intelligence. Decision-makers still read PDF reports from yesterday. BIE changes that.*

**Bharat Intelligence Engine** is a real-time, open-source geospatial intelligence platform purpose-built for the Indian subcontinent. It ingests live news feeds, government data, and open-source intelligence (OSINT); processes them through an NLP and knowledge-graph pipeline; and surfaces actionable intelligence on an interactive map-based dashboard.

---

## 🎯 Problem → Solution

| Problem | BIE's Answer |
|---|---|
| Intelligence reports are stale PDFs delivered hours late | Live ingestion pipeline with < 2-minute classification lag |
| Events lack geographic context | Every event geo-located on an interactive India-centric map |
| Analysts manually correlate signals across sources | Automated knowledge graph + convergence detection |
| No natural-language querying of intelligence databases | GraphRAG-powered Q&A with citations |
| Instability trends are subjective | Algorithmic India Instability Index (III) per region |
| Strategic briefs take hours to write | Auto-generated briefs every 6 hours via LLM |

---

## 🏗️ Tech Stack

| Layer | Technology |
|---|---|
| **Frontend** | Next.js 16, React 19, Tailwind CSS v4, MapLibre GL JS, Deck.gl |
| **Backend** | Python 3.12, FastAPI, Uvicorn |
| **NLP** | spaCy (English + Hindi/Urdu), custom India NER models |
| **LLM** | Anthropic Claude API (brief generation, GraphRAG) |
| **Knowledge Graph** | Neo4j |
| **Vector Store** | Qdrant |
| **Cache / Streams** | Redis |
| **Deployment** | Vercel (frontend) · Railway (backend) |

---

## 📁 Project Structure

```
india-innovates/
├── frontend/              # Next.js 16 web application
│   ├── app/               # App Router — pages, layouts, components
│   ├── public/            # Static assets, GeoJSON
│   ├── package.json
│   └── ...
├── backend/               # Python FastAPI service
│   ├── app/
│   │   ├── main.py        # FastAPI entrypoint
│   │   ├── routers/       # API route handlers
│   │   ├── services/      # NLP, graph, analytics services
│   │   ├── models/        # Pydantic schemas
│   │   └── workers/       # Background workers (NLP, ingestion)
│   ├── config/
│   │   └── feeds.yaml     # India Feed Registry
│   ├── scripts/           # Seeding, migration scripts
│   └── requirements.txt
├── data/                  # Seed data & static datasets
│   ├── geo/               # GeoJSON (LAC, LOC, borders, zones)
│   └── hotspots/          # India Hotspot Database
├── docs/                  # Demo script, operational docs
├── prd/                   # Product Requirements Documents
│   ├── prd.md             # Central PRD — all 42 features
│   ├── architecture.md    # Monorepo, API contracts, schemas
│   ├── backend-api.md     # FastAPI, endpoints, caching, errors
│   ├── data-ingestion.md  # Feed registry, RSS/API workers
│   ├── geospatial-frontend.md  # MapLibre, Deck.gl, layers
│   ├── ui-panels.md       # News feed, query, KG, posture, IOR
│   ├── nlp-pipeline.md    # spaCy, NER, Hindi/Urdu, classifier
│   ├── knowledge-graph.md # Neo4j, GraphRAG, Qdrant
│   ├── analytics.md       # III, anomaly, convergence, proximity
│   ├── intelligence-products.md  # Strategic brief generator
│   ├── design-system.md   # Design language, mobile
│   ├── deployment.md      # Demo env, hardening
│   └── schemas.md         # All data models
├── .gitignore
├── LICENSE                # Apache 2.0
└── README.md
```

---

## 🚀 Getting Started

### Prerequisites

- **Node.js** ≥ 18
- **Python** ≥ 3.12
- **Redis**, **Neo4j**, **Qdrant** (local or cloud instances)

### Frontend

```bash
cd frontend
npm install
npm run dev
```

Open [http://localhost:3000](http://localhost:3000) to view the app.

### Backend

```bash
cd backend
python -m venv .venv
source .venv/bin/activate   # On Windows: .venv\Scripts\activate
pip install -r requirements.txt
uvicorn app.main:app --reload --port 8000
```

API docs at [http://localhost:8000/docs](http://localhost:8000/docs).

---

## 🗺️ Feature Overview

BIE is designed around **42 features** across these modules:

| Module | Key Capabilities |
|---|---|
| **Geospatial Frontend** | MapLibre GL map, Deck.gl layers, regional presets, event markers, instability heatmap |
| **Data Ingestion** | India feed registry, RSS workers, external API workers, data freshness monitoring |
| **NLP Pipeline** | spaCy, India-specific NER, Hindi/Urdu support, hybrid threat classifier |
| **Knowledge Graph** | Neo4j schema, automated graph writer, GraphRAG query engine, Qdrant vectors |
| **Analytics Engine** | India Instability Index, anomaly detection, signal convergence, infrastructure proximity |
| **Intelligence Products** | Auto-generated strategic briefs every 6 hours |
| **UI Panels** | News feed, NL query interface, knowledge graph, strategic posture, IOR dashboard |
| **Design & UX** | Dark-mode design language, glassmorphism panels, mobile optimization |
| **Deployment** | Demo environment, scripted demo, production hardening |

> Full specifications → [`prd/prd.md`](prd/prd.md)

---

## 📋 Documentation

| Document | Covers |
|---|---|
| [`prd.md`](prd/prd.md) | Master PRD — all 42 features, critical paths, risk register |
| [`architecture.md`](prd/architecture.md) | Monorepo, API contracts, shared schemas, integration tests |
| [`backend-api.md`](prd/backend-api.md) | FastAPI foundation, endpoints, caching, error handling |
| [`data-ingestion.md`](prd/data-ingestion.md) | Feed registry, RSS worker, external APIs, freshness monitor |
| [`geospatial-frontend.md`](prd/geospatial-frontend.md) | MapLibre, Deck.gl, geo layers, presets, markers, heatmap |
| [`ui-panels.md`](prd/ui-panels.md) | News feed, NL query, KG panel, posture, IOR dashboard |
| [`nlp-pipeline.md`](prd/nlp-pipeline.md) | spaCy, India NER, Hindi/Urdu, threat classifier, NLP worker |
| [`knowledge-graph.md`](prd/knowledge-graph.md) | Neo4j, graph writer, GraphRAG, Qdrant vector store |
| [`analytics.md`](prd/analytics.md) | Hotspot DB, instability index, anomaly, convergence, proximity |
| [`intelligence-products.md`](prd/intelligence-products.md) | Strategic brief generator |
| [`design-system.md`](prd/design-system.md) | BIE design language, mobile optimization |
| [`deployment.md`](prd/deployment.md) | Demo environment, demo script, production hardening |
| [`schemas.md`](prd/schemas.md) | All data models — API, Neo4j, Qdrant, Redis |

---

## 🤝 Contributing

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

---

## 📄 License

This project is licensed under the **Apache License 2.0** — see the [LICENSE](LICENSE) file for details.

---

<p align="center">
  Built for the <strong>India Innovates</strong> hackathon 🇮🇳
</p>