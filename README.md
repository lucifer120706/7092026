# OneCode AI — Different Codes. One Material. OneCode.

SIH 2026 · Problem Statement 26099 · Ministry of Petroleum & Natural Gas / CPCL

**AI-Driven Material Standardization & Harmonization Platform.**

Different CPSE Codes → One Standard Specification → **OneCode**.

An AI-driven reconciliation layer that ingests heterogeneous CPSE material
records (SAP/ERP export, CSV, Excel, JSON, API), identifies duplicates,
near-duplicates, and functionally equivalent materials, and assigns each a
unified **OneCode** — while preserving full traceability to every originating
CPSE material code. It does not replace any CPSE's SAP/ERP system.

All data bundled with this prototype is **synthetic/demo data**, clearly
labelled as such throughout the UI. Real CPSE material master data can be
supplied via the Data Onboarding module in the same formats.

## What's new in this version

- **OneCode identity model**: transitively-linked duplicates/equivalents are
  clustered (union-find) into a single OneCode, not just pairwise matches —
  `OC-BRG-12808D` style codes, deterministic per unique specification.
- **Simple 5-way classification**: Duplicate, Equivalent, Similar, No Match,
  Insufficient Data — no black-box single similarity score.
- **Code Lookup** (forward: CPSE + code → OneCode + cross-CPSE mapping table;
  reverse: OneCode → all mapped CPSE codes).
- **Material 360°** view: identity, CPSE codes, technical attributes, AI
  decision evidence, procurement intelligence, and audit history in one place.
- **Explainable AI**: every match shows a structured evidence table (material,
  dimension, standard, critical attributes, manufacturer) — never a single score.
- **Configurable CPSE list and category ontology** (`backend/config.py`) —
  not hard-coded into the matching engine.
- **Source-agnostic Data Onboarding**: upload CSV, Excel, or JSON; columns are
  auto-detected against known field names (including SAP codes like MATNR,
  MAKTX) with an editable mapping and confidence score.
- **Data Quality scoring** per CPSE and overall.
- **CPSE Analytics, OneCode Master, CPSE (legacy code) Mapping** tables.
- **Procurement Insights**, explicitly labelled "Illustrative Demo Estimate."
- **Model Evaluation** (precision/recall/F1/false-merge rate) — only computed
  against the bundled synthetic ground truth, clearly labelled as such.
- 13-section enterprise navigation matching the full product spec.

## Repository structure

```
onecode-ai/
├── backend/
│   ├── main.py               FastAPI app — 25+ endpoints across all 13 modules
│   ├── pipeline.py            matching engine: normalize → ontology → blocking →
│   │                          hybrid scoring → 5-way decision → clustering → OneCode
│   ├── config.py               configurable CPSE list + category ontology + column synonyms
│   ├── generate_dataset.py      regenerates the bundled synthetic demo dataset
│   ├── data/sample_materials.csv
│   ├── requirements.txt          LIGHT deps — TF-IDF semantic matching (Render free tier)
│   ├── requirements-full.txt      FULL deps — adds sentence-transformers/MiniLM
│   ├── Dockerfile                 for the FULL version (Hugging Face Spaces / Railway)
├── dashboard/
│   └── onecode_ai_app.html    13-tab enterprise dashboard (plain HTML/JS, no build step)
└── notebook/
    └── onecode_final_kaggle_notebook.py   Kaggle/Colab-ready pipeline (8 cells)
```

## Running the backend locally

```bash
cd backend
pip install -r requirements.txt      # or requirements-full.txt for real embeddings
uvicorn main:app --reload --port 8000
```

Visit `http://localhost:8000/docs` for interactive Swagger UI.

## Running the dashboard

Open `dashboard/onecode_ai_app.html` directly in a browser. By default it
points at `http://localhost:8000`. To point it at a deployed backend, append
`?api=https://your-backend-url` to the page's address — no build step, no
editing required.

## Deploying

| Host | Tier | Requirements file | Real embeddings? |
|---|---|---|---|
| **Render** | Free (512MB RAM) | `requirements.txt` | No — auto-falls-back to TF-IDF |
| **Hugging Face Spaces** | Free (Docker SDK, ~16GB RAM) | `requirements-full.txt` + `Dockerfile` | **Yes** |
| **Railway** | Free trial credits | `requirements-full.txt` + `Dockerfile` | Yes |

For the dashboard: deploy `dashboard/onecode_ai_app.html` as a static site on
Vercel or Netlify (drag-and-drop, no build step), then share the link with
`?api=<your-backend-url>` appended.

### Render (backend, TF-IDF, simplest)
1. render.com → New → Web Service → connect this repo
2. Root directory: `backend`
3. Build command: `pip install -r requirements.txt`
4. Start command: `uvicorn main:app --host 0.0.0.0 --port $PORT`

### Hugging Face Spaces / Railway (backend, real embeddings)
1. Create a Space (SDK: Docker) or Railway project from this repo
2. Root directory: `backend` (it will use the included `Dockerfile`)
3. Test `/health` — should show `"semantic_method":"MiniLM"`

## Demo flow (3–5 minutes)

1. **Overview** — show national material statistics
2. **Data Onboarding** — upload a sample CSV/Excel, show automatic column detection
3. **AI Matching** — show a Duplicate and an Equivalent result with evidence
4. **Code Lookup** — CPSE=ONGC, code=ONGC-1001 → see the OneCode and every
   mapped CPSE code
5. **Material 360°** — full identity + evidence + procurement view
6. **Show a conflict**: a grade 8.8 vs 10.9 fastener pair → "No Match —
   Specification Conflict, do not merge" despite high text similarity
7. **Procurement Insights** — aggregated demand across CPSEs (illustrative estimate)

## Honesty notes for judges/mentors

- All bundled data is synthetic, clearly labelled "Demo CSV (synthetic)" in the UI.
- Semantic similarity uses TF-IDF by default (Render-safe); real
  sentence-transformers/MiniLM embeddings activate automatically when
  deployed with `requirements-full.txt` and internet access.
- Evaluation metrics (`/evaluation`) are computed only against the bundled
  synthetic ground truth and are explicitly labelled as such — they do not
  represent real CPSE performance.
- Procurement savings are labelled "Illustrative Demo Estimate," never a
  guaranteed figure.
- SAP/ERP integration in this build means **file-export-compatible
  ingestion** (CSV/Excel/JSON with SAP-style column auto-detection), not a
  live authenticated SAP connector — stated plainly in the System
  Integration tab.
