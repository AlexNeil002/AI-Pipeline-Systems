# Operational Knowledge Capture

How legacy and tacit business knowledge is captured, structured, and made usable by the pipeline.

---

## The Problem

Transaction data (POS, bookings, invoices) is only half the picture. A restaurant's revenue numbers mean something different once you know that a dish takes 45 minutes of prep, uses a supplier with a 3-day lead time, and has never been successfully batch-cooked. An AI model that sees the sales data without that context will produce recommendations that look correct and aren't.

Most AI integrations skip this layer. They ingest the transaction data and treat everything else as out of scope. This produces outputs that are technically accurate and operationally useless.

Operational knowledge capture bridges that gap — turning informal, staff-held, and legacy-format knowledge into structured data the pipeline can reason about.

---

## What Gets Captured

### 1. Menu and Prep Knowledge (Restaurant / Food Service)

The relationship between what gets sold and what it takes to produce it.

Captured through the **Kitchen Knowledge Intake**:

| Field | Purpose |
| --- | --- |
| Menu item and variants | Canonical item identity for mapping |
| Prep component | What preparation the item requires |
| Prep timing | How far in advance prep must happen |
| Shelf life | How long prepped components remain usable |
| Batch size | Minimum/optimal production unit |
| Ingredient / component links | What stock items this item draws down |
| Operational constraints | Capacity, storage, staffing, equipment limits |

This intake does not capture private recipes. It captures the **operational structure** — the relationships, timings, and constraints that govern how the kitchen actually runs.

Once captured, this becomes the foundation for:

- Demand forecasting layered through operational reality (sales → covers → dish family → component → prep/order candidate)
- Contribution margin analysis that reflects actual production cost, not just sale price
- Exception flags when forecast demand conflicts with prep capacity or supplier lead time

---

### 2. Supplier and Purchasing Knowledge

The relationship between what gets used and where it comes from.

Captured through the **Supplier Setup Intake**:

| Field | Purpose |
| --- | --- |
| Supplier inventory | Who supplies what, ordering method, cadence, lead time |
| Data access | What supplier data exists, in what format, who owns it |
| Mapping requirements | Supplier item names → internal stock items |
| Ordering rules | Minimums, lead times, delivery days, substitutes, emergency orders |
| Ordering ownership | Who currently makes each purchasing decision |

Supplier knowledge is treated as review-only in V1 unless the client already has reliable, structured cost data. Safe V1 outputs: source status, missing supplier data, unmapped items, order-history summaries, review-only ordering candidates. Auto-ordering does not happen without validated mappings, COGS data, and named approval owner.

---

### 3. Legacy Source Handling

Many SMBs hold critical operational knowledge in formats that were never designed for analysis:

- Handwritten prep sheets
- WhatsApp or email threads with supplier confirmations
- Spreadsheets maintained by one staff member
- POS category codes that mean something only to the person who set them up
- Booking notes in free text

These are treated as **intake sources**, not data sources. The intake process structures the information into the pipeline schema through guided forms, not direct import. Each item captured through intake gets a source contract, a confidence rating, and an owner before any logic runs on it.

The guiding principle: **structured intake forms beat unstructured conversation**. Operational knowledge captured in a form is reproducible, reviewable, and auditable. Knowledge captured through a chat session is not.

---

## The Intake Process

```text
Identify knowledge gap → Select intake form → Complete with client/staff → Review with approval owner → Enter into pipeline schema → Document in source contract
```

Key rules:

- Intake responses belong to the client system, not the agency system
- No private recipe, cost, or supplier detail is stored in shared or agency repositories
- Intake-derived data carries a confidence marker — information from memory or informal sources starts at low/medium until cross-checked against transaction evidence
- Mappings derived from intake go through the same review queue as any other mapping — human approval before downstream logic runs on them

---

## What Intake Enables Downstream

Without operational knowledge capture, the pipeline is limited to surface-level analysis: revenue by item, booking volume, no-show rate. Accurate but shallow.

With it, the AI layer can reason across the full operational picture:

| Without intake | With intake |
| --- | --- |
| Revenue by menu item | Revenue vs. prep cost vs. prep time per item |
| Top sellers by volume | Top sellers adjusted for kitchen capacity constraints |
| Demand forecast | Demand forecast layered through prep timing and supplier lead time |
| Contribution margin (revenue only) | Full contribution margin (revenue minus actual COGS) |
| Booking forecast | Booking forecast mapped to covers, then to dish demand, then to prep candidates |

The forecasting chain for a restaurant looks like this:

```text
Bookings → Covers → Dish family demand → Dish variant demand → Component demand → Ingredient demand → Prep candidate → Order candidate → Manager approval
```

Each step in that chain depends on intake data. Without it, the chain breaks early and outputs stay low-confidence.

---

## V1 Boundaries

In V1, intake data is used conservatively:

**Safe uses:**
- Explaining mappings between source names and canonical items
- Flagging missing operational assumptions that would affect forecast accuracy
- Building review queues for unmapped or ambiguous items
- Supporting prep-planning candidates for human review

**Not safe in V1:**
- Automating prep decisions from informal kitchen knowledge alone
- Triggering supplier orders without validated COGS, lead time data, and named approval owner
- Making menu or pricing recommendations from unvalidated intake responses

Intake data matures into higher-confidence inputs as it is cross-checked against transaction evidence over time. The pipeline tracks confidence per source and adjusts output confidence accordingly.
