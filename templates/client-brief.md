# Client Brief — AI Operational System Engagement

> Complete all sections before development begins.
> This document becomes the source of truth for the CLAUDE.md context file and the source contract library.

---

## Client Details

**Business name:**
**Vertical:** [restaurant | retail | salon | gym | trades | e-commerce]
**Owner name:**
**Contact email:**
**Date of brief:**
**Location:**

---

## Business Overview

**What does this business do?**
[2–3 sentences. Enough to understand the operation without visiting.]

**How long has it been running?**

**Approximate monthly revenue:**

**Team size:**

---

## The Operational Problem

**What decision is currently being made badly, slowly, or without data?**
[Be specific. Not "wants better data" — what is the owner doing today that this system will improve?]

**What does the owner do today without this system?**
[How are they currently getting — or not getting — this information?]

**What would change if they had daily pipeline output?**
[Concrete answer. What would they do differently on a Tuesday morning?]

---

## Data Inventory

**What data exists?**

| Source | What it contains | Format / Access method | How far back | Known quirks |
| --- | --- | --- | --- | --- |
| | | | | |

**What data is missing or uncertain?**
[COGS, customer identifiers, supplier costs, ingredient data — be honest here. Missing data affects what can be delivered at V1.]

**Data quality notes:**
[Encoding issues, inconsistent categories, manual gaps, export schedule limitations]

---

## Source Access

| Source | API available? | Export available? | Credentials needed | Contact |
| --- | --- | --- | --- | --- |
| | | | | |

---

## Desired Outputs

**What does the owner want to see from this system?**
[In their language. Specific metrics, summaries, or alerts.]

1.
2.
3.

**What should trigger an alert or exception?**
[e.g. revenue below X, no-show rate above Y%, invoice overdue by Z days, stock below threshold]

**How do they want to receive output?**

- [ ] HTML dashboard (bookmarked)
- [ ] Review pack (operator reviews before client sees output)
- [ ] Notion page
- [ ] Other: ________

---

## Approval and Review

**Who is the approval owner?** (person who signs off on review packs)

**How often should review packs be generated?**
[Weekly / per pipeline cycle / on exception]

**What decisions require human approval before any action is taken?**
[e.g. reorder triggers, campaign sends, staffing recommendations]

---

## Technical Environment

**Current tools the client uses:**
[List everything: POS system, booking platform, accounting software, communication tools]

**Who manages their tech?**
[Owner / staff member / external IT]

**Tech comfort level:**
[High / Medium / Low — affects how we design the interface layer]

**Any security or data constraints?**
[e.g. GDPR considerations, data residency, access restrictions]

---

## Engagement Scope

**What is in scope for V1?**

1.
2.
3.

**What is explicitly out of scope at V1?**
[Set this clearly before development begins.]

**What would V2 add?**
[Forward planning — not a commitment.]

---

## V1 Completion Criteria

V1 is complete when:

- [ ] Source inventory exists for every active data source
- [ ] Data-readiness audit completed
- [ ] Source contracts created for every active source
- [ ] Raw, clean, mapping, audit, and output data areas defined
- [ ] At least one useful decision supported end-to-end
- [ ] Unmapped, stale, and missing data is reviewable (not silently dropped)
- [ ] Approval owner named and first review pack signed off
- [ ] V2 next steps documented

---

## Success Definition

**How will we know this engagement was successful?**
[Specific, measurable. e.g. "Owner can see yesterday's revenue split by category and top 5 items by 8am without opening their POS system."]

---

## Notes

[Anything else relevant — client concerns, dependencies, constraints, context from the sales conversation, data gaps to resolve before V1 can complete]
