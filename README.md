# AI Pipeline Systems — Alexander William Neil

I build systems that connect real business data to AI models and deliver the output into working operations. This repo documents the methodology, architecture, and validated results behind that work.

**This is not a course project. It is active work for a paying client.**

---

## Quick Overview

| | |
| --- | --- |
| **What I do** | AI integration and operational pipeline systems for SMBs |
| **Current client** | Nok Nok Restaurant, Mumbles, Wales (Client 001) |
| **Data processed** | 46,304 POS rows + 5,000 bookings — cleaned, validated, modelled |
| **Systems live** | Revenue forecasting, contribution margin engine, no-show scoring (44/44 tests passing) |
| **Stack** | Python · dlt · Pandera · DuckDB · n8n · Claude · OpenAI · GitHub · Notion |

---

## What I Do

Most organisations trying to integrate AI have the same problem: they have data, they have access to models, and no reliable system connecting the two. The output is either not trustworthy, not usable, or both.

I build the connecting layer — data ingestion, validation, AI processing, approval gates, and client-facing output — as a system that runs repeatedly, flags its own gaps, and never surfaces a confident answer from incomplete data.

**Technical capabilities:**

- **AI model integration** — connecting structured business data to large language models (Claude, GPT) for analysis, classification, forecasting, and decision support
- **Multi-model orchestration** — managing task delegation across models using Notion as a structured task and prompt hub; assigning Codex vs. Claude Code based on task type, context requirements, and cost profile
- **Prompt engineering and output validation** — designing prompts that produce structured, testable outputs; validating model behaviour against real data at scale
- **Pipeline architecture** — data ingestion (dlt), schema validation (Pandera), local analytics (DuckDB), workflow automation (n8n)
- **AI testing and confidence scoring** — every model output carries a confidence marker; low-confidence outputs are routed to review, not surfaced as reliable
- **Approval gate design** — systems where AI prepares and humans approve; governance built in from the start, not added after

---

## Live Work — Nok Nok Restaurant (Client 001)

First complete client engagement. A restaurant in Mumbles, Wales. Used to validate the full pipeline methodology end-to-end on real operational data.

**Data cleaned and validated:**

- Square POS — 46,304 transaction rows (UTF-16 encoding, GBP handling, refund logic)
- ResOS bookings — 5,000 rows (tab-delimited, status normalisation)

**Systems built and validated:**

| System | What it does | Status |
| --- | --- | --- |
| Packet 01 — Ingest & Validate | Schema validation, import logs, missing source detection | Production-ready |
| Packet 06 — Revenue Forecast | Revenue forecasting from historical sales patterns | Live (degraded — no COGS yet) |
| CM Engine — Contribution Margin | 533 menu items classified into A/B/C revenue bands | Validated |
| No-Show Engine | Scores upcoming bookings for no-show probability; overbooking recommendations to weekly review pack | Gate 2 complete, 44/44 tests passing |

**Degraded mode in practice:** COGS data not yet received from client. Rather than blocking or overclaiming, the system switches to `revenue_only_mode=true` — outputs are confidence-marked as low, routed to operator review only, and the review pack includes an explicit COGS warning before the sign-off block. The system continues operating usefully on available data without misrepresenting what it knows.

**Client interface:** HTML dashboard (auto-refresh every 5 minutes), weekly review pack with operator approval log — bookmarked and in active use.

Full detail: [docs/verticals.md](docs/verticals.md)

---

## AI Model Orchestration

A core part of the methodology is how AI models are used together, not just individually.

**Context management:** Each client project has a `CLAUDE.md` file — a persistent context document read by Claude Code at the start of every session. It records the project state, active data gaps, decisions made, and current task. No re-explaining the project each session. Claude Code and OpenAI Codex share context through this file and through structured task records in Notion.

**Task delegation via Notion:** All tasks for a project live in Notion with structured prompts. From there, tasks are delegated to the appropriate model:

- Complex implementation and multi-step reasoning → Claude Code (with CLAUDE.md context)
- Spec generation, structured planning, step-by-step task breakdown → OpenAI Codex
- Routine drafts, summaries, review pack generation → lighter models where appropriate

This means each model handles what it is best suited for, context flows cleanly between sessions and models, and compute cost is managed deliberately rather than by default.

**Internal agency operations:** Claude Cowork (team plan) handles internal workflows — lead triage, briefing drafts, proposal preparation, invoice tracking. Same approval model: Claude prepares, I approve before anything is sent or actioned.

---

## Pipeline Architecture

Every system follows the same 8-stage shape:

```text
Raw → Validation → Clean → Mapping → Aggregation → Model → Output → Approval
```

Each stage is independently testable. Each model output carries a confidence marker. Nothing reaches the client without passing through a human approval gate.

Full breakdown: [docs/architecture.md](docs/architecture.md)

---

## Automation

**n8n (self-hosted, Docker)** — workflow orchestration: scheduled pipeline runs, file detection, exception alerts, internal status digests.

**Tiered automation model** — automation is implemented incrementally, not all at once:

| Tier | What it does | Status on live work |
| --- | --- | --- |
| 0–2 | Validation, exception detection, AI drafts | Active |
| 3–5 | Mapping suggestions, approved writes, internal updates | Active |
| 6 | Business-critical automation (orders, campaigns, external sends) | Deferred — trust earned first |

This is the approach I'd apply inside any organisation integrating AI: build from the tiers you can validate, earn trust at each level before progressing.

Full methodology: [docs/methodology.md](docs/methodology.md)

---

## Skills at a Glance

| Skill area | Specifics |
| --- | --- |
| AI model integration | Claude (Anthropic API), OpenAI (Codex / GPT); structured output design, model testing |
| Prompt engineering | Structured prompts, context management via CLAUDE.md, multi-model task delegation |
| Data pipelines | dlt (ingestion), Pandera (schema validation), DuckDB (local analytics), Python |
| Workflow automation | n8n (self-hosted Docker), scheduled workflows, file-watch triggers |
| AI governance | Tiered automation model, confidence scoring, approval gates, degraded mode handling |
| Version control | GitHub, branch-per-task, structured commit history |
| Client delivery | Source contracts, review packs, HTML dashboards, operator approval logs |
| Context management | CLAUDE.md persistent context, Notion task hub, cross-model session continuity |

---

## Stack

| Layer | Tools |
| --- | --- |
| AI models | Claude (Anthropic), OpenAI Codex / GPT |
| Language | Python |
| Data ingestion | dlt |
| Schema validation | Pandera |
| Local analytics | DuckDB |
| Workflow automation | n8n (Docker) |
| Task and context hub | Notion |
| Version control | GitHub |
| Client interface | HTML dashboards |
| Internal operations | Claude Cowork (team plan) |

---

## Repo Structure

```text
docs/methodology.md        — Development workflow, multi-model orchestration, V1 checklist
docs/architecture.md       — 8-stage pipeline breakdown, automation tiers, approval model
docs/verticals.md          — Per-vertical notes and full Nok Nok validated results
docs/knowledge-capture.md  — Operational knowledge intake: prep, suppliers, legacy sources
docs/client-system.md      — Client-facing dashboard architecture, approval chain, Notion backend
docs/agency-scaffold.md    — Repeatable scaffold, learning loop, relay system
templates/                 — CLAUDE.md context template and client brief template
```

---

*Client repos are private. This repo documents the methodology, architecture, and validated results.*
