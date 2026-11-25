# NEAR Intents Concentration Dashboard

A read-only analytics dashboard that measures how concentrated **volume and fees** are across integrating apps and chains on NEAR Intents, with a side-by-side comparison against a cross-chain competitor (THORChain or Across).

> **Thesis:** NEAR Intents revenue is real but fragile. Most fees come from a small number of integrating apps, and deposits rely on a trusted bridge.

This repo is the **Technical track** deliverable. Its outputs (metrics, charts, comparison tables) feed the **DeFi track** research memo and pitch.

---

## Background

NEAR Intents is a cross-chain swap layer. Users state the outcome they want, competing market makers (solvers) execute it, and trades settle on NEAR's Verifier contract (`intents.near`). In February 2026, NEAR activated a fee switch: Intents fees now fund NEAR token buybacks and make up most of the chain's revenue.

**Question:** Is that revenue durable?

This project answers the measurable half of that question: *how dependent is the fee stream on a few apps and routes?*

---

## Features

### 1. Concentration dashboard (core)
- Volume and fees broken down by **integrating app**, **source chain**, **destination chain**, and **asset**
- Concentration metrics over time:
  - Top-1 and top-5 share
  - HHI (Herfindahl-Hirschman Index)
  - Gini coefficient
- Before/after view around the **February 2026 fee switch**
- Date-range and granularity filters (daily / weekly)

### 2. Competitor comparison
- Same metrics computed for **THORChain** (Midgard API) or **Across** (API / Dune)
- Side-by-side charts: NEAR Intents vs. competitor
- Explicit **entity mapping doc** defining what "app" means for each protocol (see `docs/definitions.md`)

---

## Architecture

```
┌──────────────┐   ┌──────────────┐   ┌───────────────┐   ┌─────────────┐
│ Data sources │ → │  Ingestion   │ → │ Storage       │ → │ Metrics +   │
│ NEAR Intents │   │  (scheduled  │   │ (DuckDB or    │   │ Dashboard   │
│ THORChain /  │   │   scripts)   │   │  Postgres)    │   │ (Streamlit  │
│ Across       │   │              │   │               │   │  or Next.js)│
└──────────────┘   └──────────────┘   └───────────────┘   └─────────────┘
```

Read-only by design: no wallets, no transactions, no private keys.

---

## Repo structure

```
.
├── README.md
├── docs/
│   ├── definitions.md        # what counts as an "app", fee, volume, etc.
│   ├── data-sources.md       # endpoints, auth, rate limits, quirks
│   └── methodology.md        # metric formulas and caveats
├── ingestion/
│   ├── near_intents.py       # NEAR Intents Explorer API client
│   ├── competitor.py         # THORChain / Across client
│   └── schema.sql
├── metrics/
│   ├── concentration.py      # top-N share, HHI, Gini
│   └── tests/
├── dashboard/
│   ├── app.py                # entrypoint
│   └── pages/
│       ├── overview.py
│       ├── apps.py
│       ├── chains.py
│       └── comparison.py
├── data/                     # local DB / cached pulls (gitignored)
├── notebooks/                # exploration for the memo
├── .env.example
└── requirements.txt
```

---

## Getting started

### Prerequisites
- Python 3.11+
- An API key for the NEAR Intents Explorer API, if required (see `docs/data-sources.md`)

### Setup

```bash
git clone <repo-url>
cd near-intents-dashboard
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
cp .env.example .env        # add API keys if needed
```

### Ingest data

```bash
python -m ingestion.near_intents --from 2025-10-01 --to today
python -m ingestion.competitor --protocol thorchain --from 2025-10-01 --to today
```

### Run the dashboard

```bash
streamlit run dashboard/app.py
```

---

## Metrics

| Metric | Definition | Reading it |
|---|---|---|
| Top-1 share | Largest entity's share of total volume (or fees) | Dependence on a single app |
| Top-5 share | Combined share of the five largest entities | Dependence on a small cluster |
| HHI | Sum of squared percentage shares (0-10,000) | Above 2,500 is conventionally "highly concentrated" |
| Gini | Inequality of the distribution (0-1) | Higher means more unequal |

Metrics are computed per period (weekly by default) for apps, source chains, destination chains, and assets. Formulas and caveats live in `docs/methodology.md`.

---

## Key definitions (decide early, document in `docs/definitions.md`)

- **App / integrator:** how an integrating front end is identified in the data (referral ID, fee recipient, etc.) and how it maps to the competitor's equivalent
- **Volume:** USD value at time of swap, and which price source is used
- **Fees:** which fee components count (protocol fee, app fee, solver spread) and what is excluded
- **Fee-switch date:** the cutoff used for before/after comparisons

These choices drive every concentration number, so changing one means recomputing everything.

---

## Data sources

| Source | Used for | Notes |
|---|---|---|
| NEAR Intents Explorer API | Swap-level volume, fees, chains, assets, app attribution | Confirm auth and attribution fields in week 1 |
| THORChain Midgard API | Competitor swap volume and fees by pool/affiliate | Public; affiliate fees approximate "apps" |
| Across API / Dune | Alternative competitor | Depends on available fee and integrator data |

---

## Known limitations

- Concentration is only as good as **app attribution** in the source data. Unattributed volume is shown as its own bucket, not dropped.
- Cross-protocol comparisons depend on the entity mapping and are **approximate**.
- USD values depend on the pricing source and may differ from other dashboards.
- The dashboard is descriptive: it measures concentration, it does not forecast fee durability.

---

## Roadmap

| Week | Milestone |
|---|---|
| 1 | Confirm API access and attribution fields; write `definitions.md`; ingest NEAR Intents data |
| 2 | Build metrics and core dashboard pages; start competitor ingestion |
| 3 | Finish comparison page; export tables and charts for the memo |
| 4 | Polish, deploy, document methodology, support the pitch |

**Stretch ideas (not committed):** revenue-at-risk simulator ("what if the top app leaves?"), Telegram/Discord concentration alerts, bridge-dependency breakdown.

---

## Team and tracks

- **Technical track:** this repo (dashboard and data pipeline)
- **DeFi track:** research memo and pitch, using the dashboard outputs to defend or revise the thesis

| Role | Owner |
|---|---|
| Data ingestion | _TBD_ |
| Metrics and methodology | _TBD_ |
| Dashboard / frontend | _TBD_ |
| Competitor analysis | _TBD_ |
| Memo and pitch | _TBD_ |

---

## Disclaimer

Research and educational project. Nothing here is financial advice.
