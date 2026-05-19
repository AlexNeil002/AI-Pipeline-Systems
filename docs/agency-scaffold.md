# Agency Scaffold and Learning Loop

How each client engagement produces reusable assets that make the next one faster and more reliable.

---

## The Core Problem With Agency Delivery

Most agency work is not repeatable. Each engagement starts from scratch because the knowledge from the last one lives in someone's memory, an informal doc, or is too client-specific to reuse directly.

This system is built to avoid that. Every client engagement produces two things: the client system itself, and a set of sanitised, generalised assets that get promoted back into the agency scaffold. The next client benefits from every lesson learned on the previous one — without any private data crossing the boundary.

---

## The Scaffold Every Client Receives

When a new client engagement begins, they receive a project structure built from the current agency scaffold:

```text
[ClientName]_Systems_Master/
├── CLAUDE.md                          — Persistent AI context for this client
├── config/
│   └── source_contracts/              — One contract per data source
├── data/
│   ├── raw/                           — Source files, untouched
│   ├── clean/                         — Standardised tables
│   ├── mapping/                       — Canonical entity mappings
│   ├── audit/                         — Validation logs, exception queues
│   └── outputs/                       — Model outputs, review packs
├── operations/
│   ├── decisions/                     — Decision records
│   ├── approvals/                     — Approval logs
│   └── review-packs/                  — Signed review packs
├── scripts/                           — Pipeline code for this client
└── tests/                             — Validation tests
```

This structure is not optional — it is the standard. Every client system looks the same at the scaffold level. What changes is the content: their data sources, their source contracts, their mappings, their model configuration.

The consistency means any agency team member can navigate any client system immediately, and the pipeline code written for one client can be adapted for the next without reverse-engineering a different folder structure.

---

## The Learning Loop

```text
Client Operational System
        ↓
Client observation (lesson, blocker, reusable pattern)
        ↓
Sanitised relay packet (private data removed)
        ↓
Agency review
        ↓
Promoted to: template · playbook · architecture update · capability note
        ↓
Next client scaffold
```

Every lesson that makes it through this process has had private client data removed, has been reviewed against the agency architecture, and has been confirmed as useful beyond one client before it becomes part of the standard scaffold.

Lessons that do not pass that filter stay in the client system. They are not discarded — they are just not generalised prematurely.

---

## What Was Promoted From Nok Nok (Client 001)

The first restaurant engagement produced the following additions to the agency scaffold:

| Asset | What it captures |
| --- | --- |
| POS source contract template | How to document a POS data source: fields, encoding, quirks, validation rules |
| Booking source contract template | How to document a bookings source: status normalisation, cancellation logic, no-show definition |
| Kitchen knowledge intake template | Structured capture of prep structure, components, constraints, and ingredient links |
| Supplier setup intake template | Structured capture of supplier inventory, ordering rules, data access, and mapping requirements |
| Management review pack template | Format for review packs: readiness summary, sources reviewed, recommendations, blockers, sign-off block |
| Restaurant source inventory template | Standard list of data sources expected in a restaurant engagement |
| Dashboard status model | The six-layer interface standard: status, source, data health, decision, review queue, change log |
| Approved action log template | How approvals are recorded: decision, reason, execution notes, owner, date |

Every future restaurant client starts with these templates pre-filled to the level the scaffold allows. They still need to be configured for the specific client's data — but the structure, the fields, and the validation logic are already there.

---

## The Relay Packet System

Ideas do not move between client and agency systems raw. They move through **relay packets** — structured documents that translate an idea from one context into another without leaking private data or overpromising future capability.

**Client → Agency (learning direction):**

When a client engagement reveals a reusable pattern:

```text
Client observation
→ Client-side relay packet drafted (private data removed)
→ Agency reviews for generalisability
→ Classified: record only / promote now / promote later / reject
→ Promoted assets update the scaffold
```

**Agency → Client (method update direction):**

When the agency methodology evolves:

```text
Agency method update
→ Agency-side relay packet drafted
→ Reviewed before sending
→ Client system receives and records local relevance
→ Client decides: applicable now / applicable later / blocked / not relevant
```

This prevents two failure modes that otherwise happen silently:
1. Client work revealing a better pattern that the agency never captures
2. Agency developing new capabilities that clients assume are already in scope

---

## What Stays Private

The relay system has hard boundaries. The following never cross from client to agency:

- Raw transaction data, booking records, or any client exports
- Menu mappings, recipes, yields, or prep costs
- Supplier names, terms, pricing, or order history
- Staff information
- Customer records
- Private financial or commercial assumptions

What crosses is the **shape** — the structure of the solution, the field the template needed that it was missing, the validation rule that caught a real data problem, the approval boundary that prevented an incorrect action.

---

## How This Scales

Each new vertical adds to the scaffold in the same way:

| Client type | What gets added to the scaffold |
| --- | --- |
| Restaurant (done) | POS contract, bookings contract, kitchen intake, supplier intake, CM Engine, no-show scoring |
| Retail (next) | EPOS contract, inventory contract, stock-level intake, reorder logic, SKU margin model |
| Salon | Booking platform contract, stylist intake, rebooking gap model, at-risk client scoring |
| Gym | Membership platform contract, class attendance model, churn scoring |
| Trades | Job management contract, quote tracking, job profitability model |
| E-commerce | Orders contract, returns contract, ad spend contract, ROAS model |

By the time the third or fourth client is onboarded in a given vertical, the scaffold for that vertical is largely pre-configured. Onboarding time drops significantly because the source contracts, validation logic, model templates, and review pack formats already exist.

The agency's value compounds with each engagement — not because the same work is repeated, but because each engagement makes the scaffold better.

---

## Notion as the Agency Operations Layer

The agency Notion workspace provides visibility across all active client systems simultaneously:

- Acquisition pipeline — prospects, fit scores, outreach status
- Delivery status — which clients are at which phase (audit / V1 in progress / V1 complete / V2)
- Client system health — pipeline status, degraded modes, blockers, overdue actions
- Learning log — candidate lessons from current client work awaiting review
- Relay packet queue — items in transit between client and agency systems

This is not a CRM in the traditional sense. It is an operations layer — the view that lets the agency (and eventually a small team) see what is happening across all engagements without needing to open each client repo individually.

Claude Cowork provides the connectors that keep it populated: brief generation, review pack routing, task creation from pipeline events, status summaries. The agency owner approves before anything is sent or recorded outside the internal workspace.
