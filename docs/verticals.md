# SMB Vertical Implementation Notes

How the pipeline architecture applies across each target vertical.

---

## Restaurant

**Core challenge:** High transaction volume, fast inventory turnover, labour-sensitive margins.

**Data sources:**
- POS system (item-level sales, covers, time of day, payment split)
- Reservation platform (covers booked vs. walked in, cancellations, no-shows)
- Staff scheduling (if accessible — hours scheduled vs. hours traded)

**Key pipeline outputs:**
- Menu performance: top and bottom sellers by cover, by day part, by day of week
- Labour efficiency: revenue per labour hour by shift
- No-show pattern: day/time patterns driving reservation gaps
- Daily briefing: what sold, what didn't, what to run as a special tomorrow

**Insight example:** "Tuesday lunch has run at 34% of Monday lunch revenue for the past 6 weeks. Your highest-margin items — the burger and the grilled chicken — underperform at lunch. Consider a focused lunch special anchored to those two."

---

## Retail

**Core challenge:** Inventory capital tied up in slow-moving stock, reorder timing, product mix decisions.

**Data sources:**
- POS / EPOS (item, category, margin, payment method)
- Inventory system or manual stock count (current levels, last reorder dates)
- Supplier lead times (manual input or config)

**Key pipeline outputs:**
- Slow-stock alert: items below expected sell-through rate with days-of-stock remaining
- Reorder trigger: items approaching minimum threshold based on lead time
- Category performance: revenue and margin by category, week-over-week
- Daily briefing: what moved, what's stagnating, what to reorder

**Insight example:** "You have 47 units of [product] with a current sell rate of 3 per week and 3 weeks of lead time. At this rate you'll hit zero stock in 4 weeks — reorder now to avoid a gap."

---

## Salon / Beauty

**Core challenge:** Chair and stylist utilisation, rebooking gaps, peak/off-peak imbalance.

**Data sources:**
- Booking platform (appointments, stylist, service type, duration, cancellations, no-shows)
- Client records (repeat visit rate, average spend, last visit date)

**Key pipeline outputs:**
- Utilisation by stylist: booked hours vs. available hours
- Rebooking gap analysis: clients overdue for their next visit
- No-show pattern: which time slots and which client segments drive no-shows
- At-risk clients: high-value clients who haven't rebooked within expected window
- Daily briefing: today's column, gaps to fill, clients to chase

**Insight example:** "12 clients with average spend over £80 haven't rebooked in 8+ weeks. A targeted message to this group this week could recover £960 in bookings."

---

## Gym / Fitness

**Core challenge:** Membership churn, class capacity, underused off-peak slots.

**Data sources:**
- Membership platform (active/lapsed/churned, join date, membership type)
- Class booking system (bookings, attendance, cancellations, waitlist)
- Payment system (failed payments as leading churn indicator)

**Key pipeline outputs:**
- Churn risk score: members with declining attendance + payment friction
- Class capacity: consistently underbooked slots vs. consistently full slots
- Retention alert: members approaching typical churn window (45/60/90 days low activity)
- Daily briefing: attendance summary, at-risk members, class fill rates

**Insight example:** "17 members have attended fewer than 2 sessions in the past 30 days and are on month-to-month memberships. This cohort has a 68% historical churn rate at this point. Automated check-in message recommended."

---

## Trades (Plumber, Electrician, Builder, etc.)

**Core challenge:** Job profitability visibility, quote conversion, cash flow from invoicing.

**Data sources:**
- Job management platform (job status, time logged, materials, invoiced amount)
- Quote tracker (quotes sent, won, lost, conversion rate)
- Invoicing/accounting (paid, outstanding, overdue)

**Key pipeline outputs:**
- Job profitability: revenue vs. time + materials per job, by job type
- Quote conversion: win rate by job type, by customer segment, by quote size
- Outstanding invoice alert: overdue invoices with days outstanding
- Daily briefing: jobs in progress, invoices to chase, quotes to follow up

**Insight example:** "Your bathroom refits average £340 profit per job. Your kitchen refits average £90. You're currently pricing kitchen work at the same day rate but spending 40% more time on average. Repricing this job type would add ~£1,200/month at current volume."

---

## E-commerce

**Core challenge:** ROAS, product-level margin, fulfilment efficiency, return patterns.

**Data sources:**
- Orders platform (orders, items, revenue, discount applied)
- Returns data (return rate, reason, SKU-level)
- Ad platform (spend, clicks, conversions by campaign/channel)
- Fulfilment (dispatch time, carrier, delivery success rate)

**Key pipeline outputs:**
- ROAS by channel and campaign: what ad spend is actually converting
- Product margin: revenue minus COGS minus return cost, by SKU
- Return pattern: high-return SKUs and the stated reasons
- Fulfilment flags: orders outside expected dispatch window
- Daily briefing: yesterday's revenue, top products, flagged SKUs, ad performance

**Insight example:** "Your Facebook campaign for [product category] has generated £1,800 in revenue at £620 spend this week — ROAS 2.9. Your Google Shopping for the same category is running at ROAS 5.1 at lower spend. Budget reallocation opportunity."

---

## Common Threads Across Verticals

Every vertical pipeline shares the same underlying structure — only the data sources and AI prompt templates change. This means:

- New verticals can be onboarded faster using existing modules
- Lessons from one vertical (e.g. churn detection patterns from gyms) can be adapted for others (e.g. salon rebooking gaps)
- The client-facing output (Notion dashboard + daily briefing) is consistent regardless of vertical, which simplifies client onboarding
