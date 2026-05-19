# Pipeline Architecture

Component-level breakdown of the AI pipeline system design.

---

## Overview

Each pipeline is built from five modular layers. The layers are consistent across verticals — what changes is the configuration, not the structure.

```
[Data Sources] → [Ingestion] → [Processing] → [AI Layer] → [Output Layer]
```

This modularity means a new vertical can be onboarded by configuring existing components rather than rebuilding from scratch.

---

## Layer 1 — Data Sources

What the pipeline reads from. Varies by vertical but typically includes:

- **POS / transaction systems** — sales data, item-level detail, payment method
- **Booking platforms** — appointment volume, no-shows, stylist/instructor load
- **Inventory systems** — stock levels, turnover, reorder history
- **Job management tools** — job status, time logged, invoicing (trades)
- **E-commerce platforms** — orders, returns, product-level margin, ad spend
- **Spreadsheet exports** — when no API exists, structured CSV ingestion

Data source connectors are written per-vertical and reused across clients in the same vertical.

---

## Layer 2 — Ingestion

Responsible for pulling data reliably and consistently.

- Scheduled pulls (daily, hourly depending on use case)
- API authentication handled per-source
- Raw data written to a staging area before any transformation
- Basic validation at ingestion — schema check, null handling, duplicate detection

The staging area is never written to by downstream layers. Raw data is immutable once captured.

---

## Layer 3 — Normalisation and Processing

Transforms raw data into a consistent schema the AI layer can reason about.

Key operations:
- Date/time standardisation
- Currency normalisation
- Category mapping (e.g. POS category codes → readable labels)
- Aggregation (daily totals, rolling averages, period-over-period comparisons)
- Anomaly flagging (values outside expected range held for review, not silently dropped)

Output of this layer is a clean, structured dataset. No business logic lives here — just transformation.

---

## Layer 4 — AI Processing (Claude)

The intelligence layer. Takes structured data and generates insight.

Two modes of operation:

**Analysis mode** — reads a data snapshot and produces structured insight output:
- Performance summary
- Anomalies and their likely explanations
- Ranked opportunities (what to act on first)
- Risk flags (churn signals, margin compression, stock issues)

**Generation mode** — produces formatted client-facing content:
- Daily briefing text (plain language, owner-readable)
- Alert messages for Zapier routing
- Dashboard annotations

Prompts are templated per vertical and tuned per client. The AI layer does not write to any data store — it returns structured output that the output layer handles.

---

## Layer 5 — Output Layer

Distributes results to where the client actually works.

**Notion** — primary client-facing interface:
- Dashboard database updated with each pipeline run
- Insight entries tagged by date, vertical, category
- Owner can filter, search, and review history

**Zapier** — automation routing:
- Receives structured trigger payloads from pipeline
- Routes to SMS, email, Slack, or internal tools based on trigger type
- Examples: low stock alert → WhatsApp, churn risk flag → owner email, daily briefing → Notion page creation

**Logs** — internal only:
- Each pipeline run logged with status, row counts, errors
- Retained for debugging and client reporting

---

## Cross-Cutting Concerns

**Error handling** — failures at any layer are logged and surfaced, never silently swallowed. Partial runs do not overwrite previous successful output.

**Modularity** — each layer is independently testable. New data sources plug into the ingestion layer without touching processing or AI layers.

**Vertical configuration** — a config file per client defines data sources, field mappings, AI prompt templates, and output destinations. The pipeline code itself is vertical-agnostic.

**No raw data to client** — clients never receive unprocessed data. Everything surfaced has been through the AI layer and formatted for readability.

---

## Scaling Considerations

This architecture runs comfortably for a single SMB client on minimal infrastructure (a scheduled script, cloud storage for staging, API calls to Claude and Notion). It does not require a data warehouse, a streaming system, or a dedicated server.

For multi-client deployments, the config-per-client pattern means the same codebase services multiple clients with isolated data and outputs.
