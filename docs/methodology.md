# Development Methodology

How I plan, build, and ship AI operational systems for SMB clients.

---

## The Core Principle

Every system follows the same discipline: **plan exhaustively, build incrementally, approve before acting.**

AI accelerates execution. It does not replace the governance that makes outputs trustworthy. The methodology exists to ensure no output reaches a client without a human in the loop.

---

## Phase 1 — Data Audit and Scoping

Before any code runs, the client's data environment is mapped:

- What sources exist and where they live (POS, booking platform, spreadsheets, exports)
- What format each source comes in (CSV, API, manual export, UTF-16, tab-delimited)
- What data is missing, unreliable, or blocked
- What operational decisions the owner makes day-to-day
- Which of those decisions would improve with better information

The output is a **client brief** — a structured document that defines what the pipeline will ingest, process, and surface. Nothing is built before this document is complete.

Each data source gets its own **source contract** — a document that records what the source is, how it is accessed, what fields it contains, what quirks it has, and what it enables downstream. Source contracts precede logic. Always.

---

## Phase 2 — Task Planning and Model Delegation

Each pipeline component is planned as a discrete task before any implementation begins. Tasks are managed in **Notion** — the central hub that holds structured task records, prompt templates, delegation decisions, and context for every active project.

**Model selection:** Claude Code and OpenAI Codex are used interchangeably for planning and implementation. The choice is based on task type, context requirements, and cost:

- Structured spec generation, step-by-step task breakdown → Codex
- Complex multi-step implementation, reasoning over existing codebase → Claude Code
- Routine drafts, summaries, review pack generation → lighter model where appropriate

Both models receive the same project context — through the `CLAUDE.md` file and through the Notion task record. This means a task planned in Codex can be handed to Claude Code for implementation without re-explaining the project state.

The planning prompt format (used across both models):

```text
Context: [project type, data sources, prior decisions]
Task: [specific thing to build]
Constraints: [language, existing modules, integration requirements]
Output: [what I need — spec, steps, pseudocode, file structure]
```

The spec is reviewed before any editor is opened. This catches design problems before they become code problems.

---

## Phase 3 — Implementation with Persistent Context

All code is written inside Claude Code with a project-level `CLAUDE.md` file that maintains persistent context across every session and across models.

The CLAUDE.md records:

- Project goal and client vertical
- Data sources and their source contracts
- Pipeline packets built and their status
- Active branch and current task
- Key decisions and their rationale
- Known data gaps and degraded modes in effect

Claude Code reads this at the start of every session. Codex accesses the same context through Notion. There is no re-explaining the project — any model picks up where the last session left off.

---

## Phase 4 — Branch-Per-Task Version Control

Every task from the plan gets its own Git branch:

```text
main
└── feature/packet-01-ingest-validation
└── feature/packet-06-revenue-forecast
└── feature/cm-engine-revenue-mode
└── feature/no-show-scoring-gate2
```

Rules:

- One task per branch, no exceptions
- Branch name matches the task in the planning doc
- Nothing is merged to main until reviewed
- Commit messages describe what changed and why

---

## Phase 5 — Validation Gates

Every pipeline stage has a validation gate before data moves downstream:

- **Import validation log** — pass/fail per source, row counts, encoding checks, missing source flags
- **Schema validation** — field types, null handling, expected ranges (Pandera)
- **Freshness check** — source file age, last-updated timestamp
- **Confidence marker** — high / medium / low attached to every model output

If a gate fails, the pipeline halts and logs the blocker. Partial runs do not overwrite previous successful output.

**Degraded modes:** when a required data source is missing (e.g. COGS data not yet available), the pipeline switches to a degraded mode rather than failing. The degraded mode is documented, outputs are confidence-marked as low, and those outputs are blocked from auto-action until the gap is resolved.

---

## Phase 6 — Review Packs and Approval Gates

Model outputs do not go directly to the client. They go to a **review pack** first.

A review pack is a structured document that contains:

- What the model produced
- The confidence level and any active degraded modes
- Blockers that prevented full analysis
- What the operator needs to sign off on

The operator reviews and approves. The approval is logged — decision, reason, and execution notes. Only approved outputs flow downstream.

**Hard rule:** Agents prepare. Humans approve. This applies to:

- Source contract mappings
- Forecast outputs
- Contribution margin classifications
- Any recommendation that could affect staffing, ordering, pricing, or campaigns

---

## Phase 7 — Client Interface

After approval, outputs are surfaced through:

- **HTML dashboard** — auto-refreshes every 5 minutes, client-facing, accessible via bookmark
- **Notion** — review history, task tracking, approval log
- **Daily briefing** — plain-language summary formatted for the owner

Clients do not see Python, YAML, or validation logs. They see a structured output with a clear status and a sign-off record.

---

## Phase 8 — Recurring Operational Cycle

A V1 system is not a one-shot delivery. It runs on a recurring cycle:

- Source data refreshed on schedule
- Validation gates re-run on each cycle
- Review pack generated and routed to operator
- Operator approves, exceptions are queued
- Lessons from each cycle feed back into source contracts and templates

This cycle is the product. The pipeline is the infrastructure that makes it repeatable.

---

## V1 Completion Checklist

A client system is not at V1 until all of the following are true:

1. Source inventory exists for every active data source
2. Data-readiness audit completed
3. Source contracts created for every active source
4. Raw, clean, mapping, audit, and output data areas defined
5. At least one useful decision supported end-to-end
6. Unmapped, stale, and missing data is reviewable (not silently dropped)
7. Approval owner named
8. First review pack exists and has been signed off
9. V2 next steps documented

---

## Tools Summary

| Tool | Role |
| --- | --- |
| OpenAI Codex | Spec generation, structured task planning |
| Claude Code | Implementation, multi-step reasoning, persistent context via CLAUDE.md |
| Notion | Task hub — structured prompts, model delegation, cross-session context |
| Python | Core pipeline language |
| dlt | Data ingestion and loading |
| Pandera | Schema validation at every stage |
| DuckDB | Local analytical storage — no infrastructure required |
| n8n (Docker) | Workflow automation — orchestration, digests, alerts |
| GitHub | Version control, branch-per-task isolation |
| HTML dashboards | Client-facing status and output views |
| Claude Cowork | Internal agency operations (drafts, reviews, routing) |

---

## What Makes This Different

Most AI tooling advice focuses on individual prompts. This methodology treats AI as a participant in a structured operational process — with source contracts, validation gates, approval boundaries, and a recurring cycle applied the same way they would be in any production data system.

The result is systems that are maintainable, auditable, and extensible — not scripts that only work once or only make sense to the original builder.
