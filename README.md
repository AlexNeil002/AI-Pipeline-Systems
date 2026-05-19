# AI Pipeline Systems — Alex Neil

I build systems that connect real business data to AI models and deliver the output into working operations. This repo documents the methodology and architecture behind that work — the approach I use, the tools I use them with, and the results from a live client engagement.

This is not a course project. It is active work.

---

## What I Do

Most businesses trying to integrate AI have the same problem: they have data, they have a model, and no reliable system connecting the two. The output is either not trustworthy, not usable, or both.

I build the connecting layer — ingestion, validation, AI processing, approval gates, and client-facing output — as a system that can run repeatedly, flag its own gaps, and never produce a confident answer from incomplete data.

The technical work covers:

- **AI model integration** — connecting structured business data to Claude (Anthropic) for analysis, classification, and decision support
- **Prompt engineering and output validation** — designing prompts that produce structured, testable outputs; validating model behaviour against real data
- **Pipeline architecture** — data ingestion (dlt), schema validation (Pandera), local analytics (DuckDB), workflow automation (n8n)
- **Testing and confidence scoring** — every model output carries a confidence marker; low-confidence outputs are routed to review, not surfaced as reliable
- **Approval gate design** — building systems where AI prepares and humans approve, so the automation can be trusted

---

## Live Work — Nok Nok Restaurant (Client 001)

The first complete client engagement. A restaurant in Mumbles, Wales. Used to validate the full pipeline end-to-end on real operational data.

**Data ingested and cleaned:**

- Square POS — 46,304 transaction rows (UTF-16 encoding, GBP handling, refund logic)
- ResOS bookings — 5,000 rows (tab-delimited, status normalisation)

**Systems built and validated:**

| System | What it does | Status |
| --- | --- | --- |
| Packet 01 — Ingest & Validate | Schema validation, import logs, missing source detection | Production-ready |
| Packet 06 — Revenue Forecast | Revenue forecasting from historical sales patterns | Live (degraded mode — no COGS yet) |
| CM Engine — Contribution Margin | Classifies 533 menu items into A/B/C revenue bands | Validated |
| No-Show Engine | Scores upcoming bookings for no-show probability; overbooking recommendations to weekly review pack | Gate 2 complete, 44/44 tests passing |

**Degraded mode handling:** COGS data not yet received from client. Rather than blocking or overclaiming, the CM Engine and revenue forecast run in `revenue_only_mode=true` — outputs are confidence-marked as low and routed to operator review only. This is documented in the review pack with an explicit COGS warning before any sign-off block.

**Client interface:** HTML dashboard (auto-refresh every 5 minutes), weekly review pack with operator approval log, bookmarked and in active use.

Full detail: [docs/verticals.md](docs/verticals.md)

---

## Pipeline Architecture

Every system I build follows the same 8-stage shape:

```text
Raw → Validation → Clean → Mapping → Aggregation → Model → Output → Approval
```

Each stage is independently testable. Each model output carries a confidence marker. Nothing reaches the client without passing through a human approval gate.

Full breakdown: [docs/architecture.md](docs/architecture.md)

---

## Automation and Tooling

**n8n (self-hosted, Docker)** — workflow orchestration: scheduled pipeline runs, file detection, exception alerts, internal status digests. No external dependencies.

**Tiered automation model** — not all automation is equal. I use a 0–6 tier model where:

- Tiers 0–5 are active on current client work (validation, AI drafts, exception queues, approved writes)
- Tier 6 (business-critical automation) is explicitly deferred until lower tiers have proven reliable

This is the approach I'd apply inside any organisation integrating AI for the first time: start with the tiers you can validate, build trust incrementally, and only automate actions that have earned it.

---

## Stack

| Layer | Tools |
| --- | --- |
| AI models | Claude (Anthropic), OpenAI Codex |
| Language | Python |
| Data ingestion | dlt |
| Schema validation | Pandera |
| Local analytics | DuckDB |
| Workflow automation | n8n (Docker) |
| Version control | GitHub |
| Client interface | HTML dashboards, Notion |
| Internal ops | Claude Cowork (team plan) |

---

## Methodology

```text
Plan (Codex) → Build (Claude Code) → Validate (dlt + Pandera) → Review → Approve → Ship
```

Every task is planned before a line of code is written. Every data source has a source contract before any logic runs on it. Every model output is tested and confidence-marked. Branch-per-task on GitHub throughout.

Full methodology: [docs/methodology.md](docs/methodology.md)

---

## Repo Structure

```text
docs/methodology.md     — Full development workflow
docs/architecture.md    — 8-stage pipeline breakdown
docs/verticals.md       — Per-vertical implementation notes and Nok Nok detail
templates/              — CLAUDE.md and client brief templates
```

---

*Client repos are private. This repo documents the methodology, architecture, and validated results.*
