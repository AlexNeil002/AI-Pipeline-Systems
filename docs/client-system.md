# Client-Facing System

How the pipeline surfaces output to clients and how it is monitored in operation.

---

## Overview

The client-facing system is not just a dashboard. It is a structured interface that answers the management questions an owner actually needs to make decisions — and does so in a way that makes data freshness, model confidence, and approval status visible at all times.

A client should be able to look at the system and immediately know:

- What happened since the last cycle
- What needs their attention or decision
- What is recommended and why
- What has already been approved
- What is blocked and what would unblock it

If the interface cannot answer those questions clearly, it is not ready for the client.

---

## Interface Layers

Every client-facing system is built from six layers, each with a defined purpose:

### Layer 1 — Status

The current state of the system at a glance.

Possible states: `OK` · `Needs Review` · `Missing Data` · `Blocked` · `Draft` · `Approved`

Nothing is presented as final unless it has passed through the approval gate. Draft outputs are marked as draft. Blocked outputs explain what is missing and what would unblock them.

### Layer 2 — Source

What data was used and how fresh it is.

Shows:
- Latest source received per data stream
- Freshness (hours/days since last update)
- Validation state (passed / failed / partial)
- Missing sources this cycle
- Any sensitive data warnings

A client should never be looking at model output without knowing what data that output was built from and when that data arrived.

### Layer 3 — Data Health

What is in the data that requires attention.

Shows:
- Invalid rows flagged during validation
- Unmapped items queued for review
- Stale data (older than expected)
- Duplicate risk flags
- Partial mappings awaiting confirmation
- Low-confidence assumptions

Data health issues are surfaced, not hidden. An exception that is visible is manageable. An exception that is silently dropped becomes a compounding problem.

### Layer 4 — Decision

What the models produced and what they recommend.

Each decision entry shows:
- The recommendation
- What source data it was built from
- The confidence level
- The reason the recommendation exists
- The expected impact
- The risk if acted on incorrectly
- Who the approval owner is

No recommendation reaches this layer without a confidence marker. Low-confidence recommendations are held in review until the data gap that caused the low confidence is resolved.

### Layer 5 — Review Queue

What requires a human decision this cycle.

For each item in the queue, the owner can:
- Approve
- Reject
- Defer to next cycle
- Request more information
- Exclude from current scope

The queue is the primary action interface. It is not a notification — it is a structured list of decisions with enough context to act on each one without needing to go elsewhere.

### Layer 6 — Change Log

A permanent record of what happened.

Shows:
- Files loaded this cycle
- Validation checks run and their results
- Mappings changed since last cycle
- Exceptions opened and resolved
- Outputs refreshed
- Approvals recorded (what was approved, by whom, when, with what notes)

The change log makes the system auditable. If a recommendation is ever questioned, the full chain — data source, validation result, model output, approval decision — is traceable.

---

## Models Feeding the Dashboard

The dashboard is downstream of the model layer. Each section of the dashboard is fed by one or more pipeline packets:

| Dashboard section | Fed by | Confidence |
| --- | --- | --- |
| Revenue summary | Packet 06 (Revenue Forecast) | Degraded until COGS received |
| Menu item performance | CM Engine (Contribution Margin) | Revenue-only until COGS received |
| Booking and no-show risk | Decision Science — No-Show Engine | Gate 2 complete |
| Source health | Packet 01 (Ingest & Validate) | High |
| Exception queue | Packet 01 + Packet 04 (Anomaly, when built) | Per source |
| Demand forecast | Packet 02 (when built) | Not yet active |
| Inventory / reorder candidates | Packet 03 (when built) | Not yet active |

Sections fed by unbuilt or degraded packets are shown with their status and the reason they are not yet active. They are never blank without explanation.

---

## The Approval Chain

Nothing moves from model output to client dashboard without passing through a named approval owner.

```text
Model output → Review pack generated → Approval owner reviews → Signs off → Output promoted to dashboard → Approval logged
```

What always requires approval:
- Source contract mappings
- Revenue and demand forecasts
- Contribution margin classifications
- No-show scoring recommendations
- Any output where confidence = low

What the system can do without approval:
- Validate files and detect schema errors
- Build exception queues
- Generate draft review packs
- Refresh clean tables after an approved source update
- Write internal status digests

This boundary is not a limitation — it is the feature. It means the client can trust that what they see on the dashboard has been reviewed by someone who understands the business, not just produced by a model.

---

## Review Packs

Before a dashboard section is trusted, it goes through a **review pack** — a structured document generated by the pipeline and reviewed by the approval owner.

A review pack contains:
- What the model produced this cycle
- The confidence level and any active degraded modes
- Explicit warnings for missing data (e.g. COGS warning before any margin sign-off)
- What blockers prevented full analysis
- A structured sign-off block: decision, reason, execution notes

Review packs are the first useful client interface. Before a full dashboard is built or trusted, a review pack lets the operator engage with the output, ask questions, and build confidence in what the system is producing. The dashboard follows once that trust is established.

---

## Notion as the Operations Backend

Behind the client-facing HTML dashboard, **Notion** serves as the backend operations layer — used by the agency (and in future, by team members) to monitor pipeline health across active client systems.

The Notion workspace provides:

- **Pipeline status per client** — which packets are running, which are blocked, which are in degraded mode
- **Task board** — structured tasks with assigned models (Claude Code / Codex), context snapshots, and completion status
- **Review pack history** — log of every review pack generated, signed off, and actioned
- **Learning log** — observations from delivery that are candidates for promotion to agency templates
- **Relay packet queue** — items moving between client system and agency system for review

Notion is not the source of truth for data, mappings, or model outputs — those live in the client system's file structure. Notion is the operational layer that makes it manageable to run multiple client systems simultaneously without losing visibility across any of them.

The path from current state to future state:

| Phase | Interface | What it enables |
| --- | --- | --- |
| Phase 1 (now) | Local files + static HTML | Stable schema, tested workflow, no tool lock-in |
| Phase 2 | Notion workspace | Day-to-day visibility, human workflow, task management |
| Phase 3 | Notion API + n8n | Automated updates, task creation, dashboard refresh |
| Phase 4 | Custom client portal | Branded interface, role-based approvals, richer visualisation |

---

## Design Principle

Every element of the client system follows the same rule:

> Help the user understand current status, make a decision, approve an action, resolve a blocker, trust or challenge a recommendation, or know the next step.

If an element does not help with one of those outcomes, it does not belong in the interface.

Raw data stays out of client-facing screens. Model confidence is always visible. The next action and who owns it is always clear.
