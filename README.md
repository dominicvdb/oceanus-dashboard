# Oceanus Communications — Q1: Temporal Patterns

**[▶ Open the live dashboard](https://dominicvdb.github.io/oceanus-dashboard/)**
*(runs entirely in your browser via WebAssembly — the first load takes a moment)*

An interactive visual analytics dashboard for investigating radio communications in
the fictional community of Oceanus, built with [marimo](https://marimo.io/) and D3.js.
No backend: the Python runtime is compiled to WebAssembly and executes client-side.

---

## Scope of this repository

This was a three-person project for the [VAST Challenge 2025, Mini-Challenge 3](https://vast-challenge.github.io/2025/),
covering four research questions. **This repository contains only Question 1, which was
my part of the work.**

**Full team:** AmanDeep Singh · Dominic van den Bungelaar · Kim Wilmink
**This deployment:** Q1 only — authored by Dominic van den Bungelaar

Questions 2 through 4 (community detection and topic modelling, pseudonym resolution,
and the Nadia Conti case study) were the work of my teammates and are deliberately not
included here. The complete four-question dashboard, with all contributions, lives in
the original team repository:
[dominicvdb/visualanalytics-mini-challenge3](https://github.com/dominicvdb/visualanalytics-mini-challenge3).

Beyond attribution, keeping the scope narrow has a practical benefit: the full dashboard
pulls in SciPy and Plotly and executes 83 cells, which makes for an unpleasant wait in a
WebAssembly runtime. This version needs four packages and 12 cells.

## What Q1 does

The dataset is a knowledge graph of 584 intercepted radio messages exchanged between 43
entities (people, vessels, organisations) over a two-week period. Q1 asks what the
*content and timing* of those messages reveal about who was doing what, and when.

The dashboard has three views:

**Interactive Dashboard** — the main view. All 584 messages, each classified into one of
10 communication categories using `gpt-4o-mini`. Filter by category, entity, entity type,
or a suspicion threshold, and the message list, summary statistics and charts update
together.

**Category Overview** — three linked D3 visualisations: message volume per category,
per-entity breakdown stacked by category, and a date × category heatmap that exposes when
particular kinds of traffic spiked. Clicking a category filters the other two.

**Self-Message Audit** — a data quality finding. 31 messages in the source graph are
labelled as sent by an entity *to itself*, which is not meaningful. Parsing the message
text shows most of these have a different real sender, addressing the labelled entity by
name. This view reconstructs the true sender for each and quantifies how often each
entity's traffic is affected.

## Technical notes

- **Reactive dataflow** — marimo builds a DAG over the cells; changing a filter
  re-executes only the cells downstream of it, not the whole notebook.
- **Custom D3 visualisations** — the linked-brushing charts are hand-written D3,
  embedded via `mo.iframe` with the aggregated data injected as JSON. Altair is used
  for the audit charts.
- **Precomputed classification** — the LLM categorisation is cached to CSV in `public/`,
  so the deployed dashboard needs no API key and makes no external calls.
- **Serverless deployment** — `marimo export html-wasm` produces a static site running
  Python in Pyodide. Data files are resolved through `mo.notebook_location()`, so the
  same notebook works unchanged locally and in the browser. GitHub Actions rebuilds and
  deploys on every push.

## Running locally

```bash
pip install marimo pandas altair networkx
marimo run app.py
```

## Building the static site

```bash
marimo export html-wasm app.py -o dist --mode run
python -m http.server --directory dist
```

The export must be served over HTTP — opening `dist/index.html` via `file://` will not work.

## Repository layout

```
app.py                     # the Q1 dashboard (single marimo notebook)
public/                    # data files, bundled into the WASM export
  MC3_graph.json             VAST Challenge knowledge graph
  categories_v2.csv          LLM message classifications (precomputed)
.github/workflows/         # GitHub Pages deployment
```

## Data

The source data is the VAST Challenge 2025 MC3 knowledge graph. `categories_v2.csv` was
generated once by classifying all 584 messages with the OpenAI API; that script is not
included here since the output is committed and the step does not need repeating.
