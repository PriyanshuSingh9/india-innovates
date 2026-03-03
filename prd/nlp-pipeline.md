# NLP Pipeline

> Covers: F16 spaCy Pipeline Setup · F17 India Custom NER · F18 Hindi / Urdu NLP · F19 Hybrid Threat Classifier · F20 NLP Worker Integration

---

## F16 — spaCy Pipeline Setup

### What Is Being Built

The foundational NLP processing pipeline using spaCy, configured with English language models and custom pipeline components. It processes every ingested text item — extracting entities, detecting language, tokenizing, and preparing text for downstream classification and graph writing.

### Why It Is Needed

Raw ingested text is unstructured. BIE needs to extract: **Who** is involved (entities), **Where** it happened (locations → coordinates), **What type** of event (classification), and **How severe** (threat assessment). spaCy is chosen for speed (~50 docs/sec on CPU), pipeline architecture, and built-in NER. The critical path `F16 → F20 → F22 → F23` means this feeds everything downstream.

### What Use It Provides

- Entity extraction (people, orgs, locations, dates)
- Tokenization & dependency parsing
- Language detection for Hindi/Urdu routing
- Pipeline extensibility for custom NER (F17) and classifier (F19)

### How It Is Built

1. **Install**: `pip install spacy && python -m spacy download en_core_web_lg`
2. **Pipeline class** in `backend/app/services/nlp/pipeline.py`:
   - Initialize `en_core_web_lg` model
   - `process(text)` method: runs NER, extracts entities, geocodes locations via India hotspot DB (F14)
   - Batch processing via `nlp.pipe(texts, batch_size=20)` for throughput
   - Language detection via `langdetect` to route Hindi/Urdu to F18 pipeline
3. **Target**: < 100ms per document processing time

---

## F17 — India Custom NER

### What Is Being Built

A custom Named Entity Recognition model trained to recognize India-specific entities that spaCy's default models miss: Indian military units, defence organizations, border landmarks, weapon systems, and infrastructure names.

### Why It Is Needed

spaCy recognizes "Pentagon" but not "DRDO", "Pangong Tso", "3 Bihar Regiment", "BrahMos", or "Daulat Beg Oldi airstrip". Missing these entities means BIE can't geo-locate events accurately or populate the knowledge graph with Indian-context entities.

### What Use It Provides

- Custom entity types: `MILITARY_UNIT`, `WEAPON_SYSTEM`, `INFRASTRUCTURE`, `BORDER_LANDMARK`, `INDIAN_ORG`
- Accurate geo-location for Indian border landmarks
- Richer knowledge graph connections

### How It Is Built

1. **Training data**: 200+ annotated examples from Indian defence news
2. **Training**: add custom labels to spaCy NER, fine-tune with `nlp.update()` for 30 epochs
3. **Save**: `nlp.to_disk("models/india_ner")`
4. **Evaluation**: target F1 > 0.80 on 50-example test set

---

## F18 — Hindi / Urdu NLP

### What Is Being Built

A multilingual NLP pipeline supporting Hindi and Urdu text processing — tokenization, NER, and basic classification — enabling BIE to ingest Hindi-language news (Dainik Jagran, Amar Ujala) and Urdu sources.

### Why It Is Needed

Significant Indian defence news breaks in Hindi before English. Urdu sources provide Pakistan-perspective intelligence. The risk register accepts 75% accuracy — Hindi is a bonus feature, not a blocker.

### What Use It Provides

- Hindi/Urdu text processing and entity extraction
- Transliteration to Latin script for graph consistency ("नरेंद्र मोदी" → "Narendra Modi")
- Combined output merges with English entities in the same knowledge graph

### How It Is Built

1. Use spaCy's multilingual model (`xx_ent_wiki_sm`) or `ai4bharat/IndicNER`
2. Transliteration via `ai4bharat-transliteration` library
3. Quality threshold: skip low-confidence entities rather than adding noise

---

## F19 — Hybrid Threat Classifier

### What Is Being Built

A text classifier assigning every item two labels — **Category** (MILITARY/DIPLOMATIC/ECONOMIC/INTERNAL/MARITIME) and **Threat Level** (LOW/MEDIUM/HIGH/CRITICAL) — using a hybrid rule-based + ML approach.

### Why It Is Needed

Category and threat level drive map marker colors, news feed badges, instability index scores, and heatmap intensity. Without classification, all events look identical. Critical path: `F19 → F25 → F30`.

### What Use It Provides

- Automatic categorization and threat assessment per item
- Confidence scores (low-confidence items flagged for review)
- < 10ms classification per item

### How It Is Built

1. **Layer 1 — Rules**: keyword dictionaries for each category/threat level (high precision)
2. **Layer 2 — ML**: TF-IDF + Logistic Regression for ambiguous cases
3. **Hybrid logic**: try rules first (confidence 0.95). If no match, use ML model
4. **Training**: 500 manually labeled Indian defence headlines. Target 85% accuracy.

---

## F20 — NLP Worker Integration

### What Is Being Built

A background worker consuming raw items from `feeds:raw` Redis Stream, processing each through the NLP pipeline (F16–F19), and publishing enriched items to `feeds:processed`.

### Why It Is Needed

The NLP pipeline is functions; the ingestion workers produce raw items. The NLP Worker is the glue — the service running continuously connecting them. Without it, raw items pile up unprocessed.

### What Use It Provides

- Continuous 24/7 processing within seconds of ingestion
- Enriched output: raw text → entities, category, threat level, geo-coordinates
- Redis Stream consumer groups for at-least-once processing
- Batching (20 items) for pipeline efficiency
- Dead-letter queue for failed items

### How It Is Built

1. **Worker service** in `backend/app/workers/nlp_worker.py`:
   - `xread` from `feeds:raw` stream (batch of 20, 5s block)
   - Process each item through NLP pipeline + classifier
   - Publish enriched item to `feeds:processed` stream
   - `xack` on success, dead-letter on failure
2. **Consumer groups**: Redis consumer groups enable parallel worker instances
3. **Metrics**: processing rate, error rate, average time exposed via `/api/workers/nlp/status`
4. **Launch**: separate process alongside FastAPI server

---

← Back to [prd.md](./prd.md)
