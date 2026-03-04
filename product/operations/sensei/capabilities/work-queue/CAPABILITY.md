# Capability: Work Queue

**Product**: Sensei — [PRODUCT](../../PRODUCT.md)
**Portfolio**: Operations
**Product Owner**: TBD (Operations PO)
**Status**: 📝 Draft — @FEATURE decomposition pending
**Last Updated**: 2026-03-04

---

## Business Function

Present field staff with a prioritized, grouped work queue optimized for processing 300–500 customers per CO per day — organized into action-type buckets with priority sub-groups, supporting both careful one-by-one processing and high-speed rapid-fire mode for experienced COs.

## Why It Exists (First Principles)

- **Cognitive Load**: A flat list of 300 tasks is overwhelming. Grouping by action type reduces cognitive switching (all calls together, all visits together, all admin tasks together).
- **Speed**: Collection officers must process tasks rapidly. The UI must minimize clicks, pre-load context, and provide quick outcome entry.
- **Prioritization**: Not all tasks are equal. Overdue items, high-DPD customers, and contact-blocked customers all need different handling. The queue must surface the most important work first.

---

## Feature Inventory

| Feature | Status | Description |
|---------|--------|-------------|
| Grouped Action Buckets | Draft | Tasks organized into: 📞 Calls, 🏠 Visits, 📋 Admin, 📦 External buckets |
| Priority Sub-groups | Draft | Within each bucket: Overdue, High Priority (DPD > 60), Normal, Renewal, Scheduled Callbacks |
| One-by-One Processing Mode | Draft | Primary mode: select task, view full context, execute, record outcome, return to queue |
| Rapid-Fire Processing Mode | Draft | Extended mode: single-card focus, large outcome targets, auto-advance, progress bar |
| Daily Contact Limit Enforcement | Draft | Auto-skip tasks when customer's daily contact limit reached; visual indicator |
| Queue Overview Header | Draft | Top-level summary: total tasks, completed today, overdue count, contact-blocked count |

---

## Business Rules

### Queue Structure

| Bucket | Contents | Primary Action |
|--------|----------|----------------|
| 📞 Calls | All call tasks, grouped by priority | Start Queue ▶ |
| 🏠 Visits | All visit tasks, grouped by priority | Plan Route 📍 |
| 📋 Admin | Admin tasks (reports, policy reads) | View All |
| 📦 External | Tasks from external systems (Onigiri, Matcha) | View All |

### Priority Sorting Within Buckets

1. 🔴 Overdue (SLA exceeded)
2. ⚡ High Priority (DPD > 60, playbook urgency)
3. 📋 Normal (DPD 30–60, standard flow)
4. 🔔 Renewal / Special (insurance renewal, KYC expiry)
5. 📞 Scheduled Callbacks (customer requested callback at specific time)

### One-by-One Processing Steps

1. Select task from queue
2. View full context: customer name, phone, loan summary, DPD, contact history, previous notes
3. Execute action (make call, conduct visit, complete admin task)
4. Record outcome from action's defined outcome list
5. Fill conditional fields required by outcome (e.g., PTP → amount + date)
6. Save and return to queue; completed task removed; next task appears

### Rapid-Fire Processing Mode Rules

- Triggered by "Start Queue ▶" on a bucket
- One customer at a time, full context pre-loaded
- Large tap targets for common outcomes
- "Save → Next ▶" auto-advances to next customer
- Progress bar shows "X of Y" with completion percentage
- Real-time contact limit display; blocks further calls if limit reached
- Exitable at any time to return to standard queue view

### Daily Contact Limit Rules

- Maximum contacts per customer per day: configurable (default: 2)
- System enforces limit — task is auto-skipped if limit reached
- Visual indicator shows "⚠️ Contact limit reached" on task card
- DaVinci owns the contact count data and emits ContactLimitReached event
- Sensei consumes the event and enforces the skip (see Contact Compliance capability)

---

## NFRs

| NFR | Requirement |
|-----|-------------|
| Target throughput | Queue UX must support 300–500 task completions per CO per day |
| Contact limit enforcement | System must not allow CO to initiate contact when limit reached |
| Pre-loaded context | Task details must load without additional navigation steps |
