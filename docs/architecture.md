# Pipeline Architecture

Component-level breakdown of the AI operational system design.

---

## Overview

Every client system is built from the same eight-stage pipeline. The stages are consistent across verticals — what changes is the configuration: data sources, field mappings, model parameters, and output destinations.

```text
Raw → Validation → Clean → Mapping → Aggregation → Model → Output → Approval
```

This means a new vertical can be onboarded by configuring existing components rather than rebuilding from scratch.

---

## Stage 1 — Raw

Source data lands exactly as received. No modification, no transformation.

- POS exports (CSV, XLSX — may be UTF-16, tab-delimited, or contain encoding quirks)
- Booking platform exports (CSV, API response)
- Inventory and supplier data (manual export or structured CSV)
- Each source is timestamped on arrival and stored separately

Raw data is immutable once captured. Downstream stages read from it but never write back to it.

---

## Stage 2 — Validation

The pipeline checks each source before any processing begins.

Outputs from this stage:

- **Import validation log** — pass/fail per source, row counts, encoding result, date range covered
- **Missing sources list** — sources expected but not received this cycle
- **Flagged rows** — records outside expected schema, held for review rather than silently dropped
- **Blocked sources log** — sources that failed validation and cannot proceed downstream

Implementation: **dlt** for loading, **Pandera** for schema enforcement.

If validation fails for a source, that source is blocked. The pipeline continues with remaining sources in degraded mode rather than halting entirely.

---

## Stage 3 — Clean

Transforms validated raw data into a consistent, queryable schema.

Key operations:

- Date and time standardisation
- Currency and value normalisation (GBP symbol handling, refund sign convention)
- Status field normalisation (booking statuses mapped to canonical values)
- Load timestamp preserved for traceability — every clean row knows which import cycle it came from
- No business logic at this stage — only structural transformation

Output: clean tables stored in **DuckDB** (local analytical store, no external infrastructure required).

---

## Stage 4 — Mapping

Resolves source-specific identifiers to canonical business entities.

- Source item names → canonical menu/product categories
- Source booking types → canonical service types
- Unmapped items are queued for review, not silently excluded
- Each mapping carries a confidence score
- Partial mappings are flagged — items partially matched but requiring operator confirmation

Mapping is a review workflow, not a one-time cleanup step. New items appear regularly as menus change, products are added, or booking types shift. The unmapped queue is part of the recurring operational cycle.

**Hard boundary:** mappings are reviewed and approved by a human before any downstream logic runs on them.

---

## Stage 5 — Aggregation

Builds the structured datasets that models operate on.

- Daily, weekly, and period-over-period revenue totals
- Item-level and category-level sell-through rates
- Booking volume, cancellation rates, no-show rates by time slot and day
- Rolling averages and trend indicators

Output: aggregated tables in DuckDB, queryable by model layer.

---

## Stage 6 — Model (Pipeline Packets)

Business logic applied to clean, aggregated data. Each model is a discrete **pipeline packet** — independently runnable, versioned, and testable.

Current packets:

| Packet | Function | Status |
| --- | --- | --- |
| Packet 01 | Source ingestion and validation | Validated |
| Packet 02 | Demand forecasting | Planned |
| Packet 03 | Inventory reorder triggers | Planned |
| Packet 04 | Anomaly and exception detection | Planned |
| Packet 05 | Churn and retention scoring | Planned |
| Packet 06 | Revenue forecasting | Implemented |
| CM Engine | Contribution margin classification | Validated (revenue-only mode) |
| Decision Science | No-show scoring, overbooking recommendations | Gate 2 implemented |

Every packet output carries a **confidence marker** (high / medium / low). Low-confidence outputs are routed to review and blocked from auto-action.

**Degraded modes:** when required data is unavailable (e.g. COGS data missing), packets switch to a documented degraded mode rather than failing or overclaiming.

Example: CM Engine with no COGS data runs in `revenue_only_mode=true`. Outputs are classified by revenue band only, confidence is marked low, and the review pack includes an explicit COGS warning before any sign-off block.

---

## Stage 7 — Output

Processed model results distributed to review and client interfaces.

**Review pack** — the primary output for any new or low-confidence result:

- Structured Markdown document, auto-generated per cycle
- Contains: model output, confidence level, active degraded modes, blockers, sign-off block
- Operator signs off before any output reaches the client dashboard
- Approval logged with decision, reason, and execution notes

**HTML dashboard** — client-facing interface:

- Static HTML, auto-refreshes every 5 minutes
- Sections: daily summary, exceptions, approval history, data freshness status
- No raw data exposed — all outputs have passed the approval gate
- Accessible via client bookmark

**Exception queue** — items requiring operator attention:

- Validation failures, unmapped items, anomaly flags, low-confidence outputs
- Visible in dashboard and review pack
- Not auto-resolved

---

## Stage 8 — Approval

Every significant output goes through a named approval owner before any action.

What requires human approval:

- Source contract mappings
- Revenue and demand forecasts
- Contribution margin classifications
- No-show scoring recommendations
- Supplier reorder triggers (when implemented)
- Any output with confidence = low

What agents can do without approval:

- Validate files, detect schema errors, build exception queues
- Generate draft review packs
- Refresh clean tables after an approved source update
- Write internal status digests

Approval is logged permanently: what was approved, by whom, when, and with what notes.

---

## Automation Layer (n8n)

Workflow orchestration runs in **n8n**, self-hosted via Docker at `localhost:5679`.

n8n handles:

- Detecting new source files and triggering validation
- Scheduling pipeline refresh cycles
- Generating and routing internal status digests
- Sending exception alerts for operator review
- Writing output logs and digest files

n8n boundaries — it does not:

- Read raw client data beyond file-presence checks
- Make any client-facing writes without approval
- Trigger business actions (orders, campaigns, staffing)
- Send external messages without human review

The first live n8n workflow checks Nok Nok report freshness and agency dashboard update time, then writes a local status digest. Tier 6 automation (business-critical, external writes) is deferred until trust is established through lower tiers.

---

## Internal Agency Operations

**Claude Cowork** (team plan) handles internal agency workflows:

- HubSpot connector: lead triage, outreach drafts, follow-up queues
- Google Workspace: briefings, review packs, meeting notes
- DocuSign: approval-ready proposals and agreements
- Notion MCP: Agency Command Hub access
- QuickBooks: invoice tracking and cash flow

Same approval model applies internally: Claude prepares, agency owner approves before any send, post, or payment action.

---

## Cross-Cutting Principles

**No silent failures.** Every stage logs what it processed, what it rejected, and why. Nothing is dropped without a record.

**Confidence is always visible.** Every model output carries a marker. Low-confidence outputs are never presented as reliable.

**Degraded is better than wrong.** When data is missing, the system operates in a documented reduced capacity rather than producing high-confidence outputs on incomplete inputs.

**Modularity.** Each packet and stage is independently testable. A new data source plugs into the validation stage without touching the model layer.

**No vendor lock-in.** DuckDB runs locally. n8n is self-hosted. HTML dashboards have no external dependencies. The system can move without rebuilding from scratch.
