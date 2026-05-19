# Development Methodology

How I plan, build, and ship AI pipeline systems for SMB clients.

---

## The Core Principle

Every pipeline project follows the same discipline: **plan exhaustively, build incrementally, ship cleanly.**

AI tools accelerate execution — they do not replace thinking. The methodology exists to ensure the thinking happens first.

---

## Phase 1 — Discovery and Scoping

Before anything is built, the client data environment is mapped:

- What data exists and where it lives (POS, booking platform, spreadsheets, APIs)
- What data is missing or unreliable
- What decisions the owner actually makes day-to-day
- Which of those decisions would benefit from better information

The output is a **data brief** — a Notion document that defines what the pipeline will ingest, process, and surface. This becomes the source of truth for the entire engagement.

Nothing is built before this document exists.

---

## Phase 2 — Planning with OpenAI Codex

Each feature or component of the pipeline is planned as a discrete task using OpenAI Codex before any implementation begins.

The planning prompt format:

```
Context: [project type, data sources, prior decisions]
Task: [specific thing to build]
Constraints: [language, existing modules, integration requirements]
Output: [what I need — spec, steps, pseudocode, file structure]
```

Codex returns structured implementation steps. These become the task spec. The spec goes into Notion before opening an editor.

This catches design problems before they become code problems.

---

## Phase 3 — Implementation with Claude Code

All code is written inside Claude Code with a project-level `CLAUDE.md` file that maintains persistent context across sessions.

The CLAUDE.md records:

- Project goal and client vertical
- Data sources and schema decisions
- Modules already built
- Modules in progress
- Key decisions and their rationale
- Current branch and active task

Claude Code reads this at the start of every session. There is no re-explaining the project — it picks up where it left off.

---

## Phase 4 — Branch-Per-Task Version Control

Every task from the Codex plan gets its own Git branch:

```
main
└── feature/ingestion-pos-api
└── feature/normalise-transaction-schema
└── feature/claude-insight-generation
└── feature/notion-dashboard-output
```

Rules:
- One task per branch, no exceptions
- Branch is named to match the task in Notion
- Nothing is committed to main until reviewed
- Commit messages describe what changed and why

This keeps the history clean and makes rollback trivial.

---

## Phase 5 — Output and Client Handoff

Pipeline outputs are surfaced through:

- **Notion dashboard** — client-facing, updated daily or on trigger
- **Zapier automations** — triggered actions based on pipeline output (alerts, messages, data writes)
- **Summary briefing** — plain-language daily summary, formatted for the owner

Clients do not see Python. They see a dashboard and a daily briefing. That is the product.

---

## Tools Summary

| Tool | Role in Workflow |
|---|---|
| OpenAI Codex | Task planning, spec generation |
| Claude Code | Implementation, code generation, session context |
| CLAUDE.md | Persistent AI memory across sessions |
| GitHub | Version control, branch-per-task isolation |
| Notion | Command hub, task tracking, client dashboards |
| Zapier | Automation layer between pipeline and client tools |
| Python | Core pipeline language |

---

## What Makes This Different

Most AI tooling advice focuses on individual prompts. This methodology treats AI as a participant in a structured development process — with planning, context, version control, and review applied the same way they would be for any software project.

The result is pipelines that are maintainable, auditable, and extensible — not one-shot scripts that only the original builder can understand.
