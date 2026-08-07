# Oceanus Communications — Visual Analytics Dashboard

**[▶ Open the live dashboard](https://dominicvdb.github.io/oceanus-dashboard/)**
*(runs entirely in your browser via WebAssembly — first load takes ~30 seconds)*

An interactive visual analytics dashboard for investigating radio communications in
the fictional community of Oceanus, built for the [VAST Challenge 2025, Mini-Challenge 3](https://vast-challenge.github.io/2025/).

Built with [marimo](https://marimo.io/) and D3.js. No backend — the entire Python
runtime is compiled to WebAssembly and executed client-side.

**Team:** AmanDeep Singh · Dominic van den Bungelaar · Kim Wilmink

---

## What it does

The dataset is a knowledge graph of 584 intercepted radio messages between 43 entities
(people, vessels, organisations) over a two-week period. The dashboard supports four
lines of investigation:

| | |
|---|---|
| **Q1 — Intent** | 584 messages classified into 10 communication categories using `gpt-4o-mini`; explored through linked D3 bar charts and a date × category heatmap |
| **Q2 — Timeline** | Interactive timeline of events, filterable by entity and date, to reconstruct the sequence of activity |
| **Q3 — Pseudonyms** | Similarity analysis over communication patterns to identify which codenames (`Boss`, `The Middleman`, `Mrs. Money`) map to which real entities |
| **Q4 — Nadia Conti** | Focused case study on a single entity, combining topic models (BERTopic + UMAP) with the communication network |

## Technical notes

- **Reactive dataflow** — marimo builds a DAG over 80+ cells; changing a filter
  re-executes only the downstream cells that depend on it.
- **Custom D3 visualisations** — the linked-brushing charts are hand-written D3,
  embedded via `mo.iframe` with the aggregated data injected as JSON.
- **Precomputed heavy steps** — LLM classification and topic modelling are cached to
  CSV in `public/`, so the deployed dashboard needs no API keys and no GPU.
- **Serverless deployment** — `marimo export html-wasm` produces a static site that
  runs Python (pandas, NumPy, SciPy, NetworkX, Altair, Plotly) in Pyodide.
  Deployed automatically to GitHub Pages on every push.

## Running locally

```bash
pip install -r requirements.txt
marimo run combined_app_final.py
```

Data files are resolved through `mo.notebook_location()`, so the same notebook works
unchanged both locally and in the browser.

## Building the static site

```bash
marimo export html-wasm combined_app_final.py -o dist --mode run
python -m http.server --directory dist
```

The export must be served over HTTP — opening `dist/index.html` directly via `file://`
will not work.

## Repository layout

```
combined_app_final.py    # the dashboard (single marimo notebook)
public/                  # data files, bundled into the WASM export
  MC3_graph.json           VAST Challenge knowledge graph
  categories_v2.csv        LLM message classifications (precomputed)
  topic_*.csv              BERTopic / UMAP outputs (precomputed)
scripts/
  intent_modeling.py     # one-off LLM classification (OpenAI API)
  save_topic_cache.py    # one-off topic model caching
.github/workflows/       # GitHub Pages deployment
```

### Regenerating the cached artefacts

Neither step is needed to run the dashboard.

```bash
# LLM classification — requires an OpenAI API key
pip install openai && python scripts/intent_modeling.py

# Topic model
pip install bertopic sentence-transformers umap-learn && python scripts/save_topic_cache.py
```
