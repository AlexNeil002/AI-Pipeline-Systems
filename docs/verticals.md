# SMB Vertical Implementation Notes

How the pipeline architecture applies across each target vertical, and what has been learned from real delivery.

---

## Restaurant — Validated (Client 001: Nok Nok, Mumbles)

The first real client engagement. Used to validate the full pipeline methodology end-to-end.

**Data sources used:**

- Square POS exports — 46,304 rows cleaned, UTF-16 encoding, GBP symbol handling, refund logic documented in source contract
- ResOS bookings — 5,000 rows cleaned, tab-delimited, status normalisation required

**Pipeline packets validated:**

- Packet 01 (Ingest/Validation) — production-ready, both sources documented with full source contracts
- Packet 06 (Revenue Forecasting) — implemented; running in degraded mode (no COGS yet), confidence = low, review-only
- CM Engine — validated; 533 menu items classified into A/B/C revenue bands (A: 17, B: 40, C: 322, excluded: 154); revenue-only mode active
- No-Show Engine (Decision Science Gate 2) — implemented; scores upcoming bookings for no-show probability from 35 confirmed no-shows and 578 cancellations across 5,000 bookings; outputs overbooking recommendations to weekly review pack; 44/44 tests passing

**Key outputs:**

- Revenue forecasting (degraded, review-only until COGS available)
- Contribution margin bands by menu item
- No-show probability scores for upcoming bookings
- Weekly review pack with operator sign-off block
- HTML dashboard with auto-refresh, bookmarked by client

**Active data gaps:**

- COGS data not yet received — blocks full margin analysis; revenue-only mode in effect across CM Engine and Packet 06
- Customer identifiers not available — blocks churn scoring (Packet 05)
- Menu CSV export not yet available — mapping partially manual

**What Nok Nok taught the agency:**

1. Source contracts must be written before any logic runs on the data — not documentation added after
2. The 8-stage pipeline works end-to-end on real restaurant data
3. Mapping is a recurring workflow, not a one-time task — new items appear as menus change
4. Forecasting must be layered with confidence visibility — not direct signal-to-action
5. Degraded modes must be explicit and documented — revenue_only_mode=true prevents overclaiming when COGS is missing
6. Review packs are often the right first interface — not dashboards
7. Approval gates prevent autonomous action on low-confidence or incomplete outputs
8. Structured intake forms beat unstructured conversation for capturing data like recipes, yields, and supplier details
9. The recurring operational cycle (refresh → validate → review → approve) is the actual product
10. Tier 6 automation should wait — the value of Tiers 1–5 needs to be proven first

---

## Retail

**Core challenge:** Inventory capital tied up in slow-moving stock, reorder timing, product mix decisions.

**Expected data sources:**

- POS / EPOS (item, category, margin, payment method)
- Inventory system or structured stock count (current levels, last reorder dates)
- Supplier lead times (manual input or config)

**Planned pipeline outputs:**

- Slow-stock alert — items below expected sell-through rate with days-of-stock remaining
- Reorder trigger — items approaching minimum threshold, adjusted for lead time
- Category performance — revenue and margin by category, week-over-week
- Contribution margin by SKU (requires COGS data from supplier)

**Packet dependencies:** Packet 01 → Packet 03 (inventory reorder) → CM Engine

**Known constraint:** Reorder triggers (Packet 03) require clean supplier lead time data. Without it, the packet runs in advisory mode only.

---

## Salon / Beauty

**Core challenge:** Chair and stylist utilisation, rebooking gaps, peak/off-peak imbalance.

**Expected data sources:**

- Booking platform (appointments, stylist, service type, duration, cancellations, no-shows)
- Client records (visit frequency, average spend, last visit date — if available)

**Planned pipeline outputs:**

- Utilisation by stylist — booked hours vs. available hours
- Rebooking gap analysis — clients overdue within their expected revisit window
- At-risk clients — high-value clients who haven't rebooked
- No-show pattern — time slots and client segments driving no-shows (shares logic with No-Show Engine from restaurant vertical)

**Packet dependencies:** Packet 01 → Decision Science (No-Show Engine, adapted) → Packet 05 (retention scoring)

**Known constraint:** Retention scoring (Packet 05) requires stable customer identifiers. Many salon booking platforms use email or phone as the identifier — needs source contract review per client.

---

## Gym / Fitness

**Core challenge:** Membership churn, class capacity optimisation, underused off-peak slots.

**Expected data sources:**

- Membership platform (active/lapsed/churned, join date, membership type, payment status)
- Class booking system (bookings, attendance, cancellations, waitlist)
- Payment system (failed payments as leading churn indicator)

**Planned pipeline outputs:**

- Churn risk score — members with declining attendance and payment friction
- Class capacity analysis — consistently underbooked vs. consistently full slots
- Retention alert — members approaching typical churn window (45/60/90-day low-activity flags)

**Packet dependencies:** Packet 01 → Packet 05 (churn/retention)

**Note:** Churn scoring accuracy improves significantly with 6+ months of attendance history. New client onboarding with limited history will produce low-confidence scores initially.

---

## Trades (Plumber, Electrician, Builder, etc.)

**Core challenge:** Job profitability visibility, quote conversion tracking, cash flow from invoicing.

**Expected data sources:**

- Job management platform (job status, time logged, materials, invoiced amount)
- Quote tracker (quotes sent, won, lost — often manual or in a job management tool)
- Invoicing / accounting system (paid, outstanding, overdue)

**Planned pipeline outputs:**

- Job profitability — revenue vs. time and materials per job, by job type
- Quote conversion rate — win rate by job type and quote size
- Outstanding invoice alert — overdue invoices with days outstanding and escalation flag

**Packet dependencies:** Packet 01 → CM Engine (job-type margin, not menu items) → Packet 04 (anomaly — unusually long jobs, zero-margin jobs)

**Known constraint:** Job management data quality varies significantly by trades client. Many use paper-based or informal systems. Source contract review and potential manual data entry phase required before pipeline can run reliably.

---

## E-commerce

**Core challenge:** ROAS accuracy, product-level margin, return patterns, fulfilment efficiency.

**Expected data sources:**

- Orders platform (orders, items, revenue, discount applied, fulfilment status)
- Returns data (return rate by SKU, reason codes)
- Ad platform exports (spend, clicks, conversions by campaign and channel)
- Fulfilment / carrier data (dispatch time, delivery success rate)

**Planned pipeline outputs:**

- ROAS by channel and campaign — revenue attributed vs. spend
- Product margin — revenue minus COGS minus return cost, by SKU (requires COGS data)
- Return pattern — high-return SKUs with reason code analysis
- Fulfilment flags — orders outside expected dispatch window

**Packet dependencies:** Packet 01 → CM Engine (SKU margin) → Packet 04 (anomaly — ROAS drops, return spikes) → Packet 06 (revenue forecast by channel)

**Note:** ROAS attribution accuracy depends on platform export quality. Multi-channel attribution (Google + Meta + email) requires careful source contract documentation to avoid double-counting.

---

## Common Patterns Across Verticals

All six verticals share the same pipeline shape. What changes is configuration:

- **Source contracts** — documenting the specific quirks of each client's data format
- **Mapping tables** — translating source-specific labels into the canonical schema
- **CM Engine mode** — full margin analysis (if COGS available) or revenue-only (degraded)
- **Confidence thresholds** — what confidence level triggers a review-only flag vs. auto-proceed
- **Review pack format** — sections and sign-off blocks adapted to the vertical's decision context

Lessons from one vertical transfer to others. The no-show scoring logic built for Nok Nok (restaurant) applies directly to salon appointment gaps and gym class no-shows — different data shapes, same underlying decision pattern.
