# Capability: Performance Dashboard

**Product**: Sensei — [PRODUCT](../../PRODUCT.md)
**Portfolio**: Operations
**Product Owner**: TBD (Operations PO)
**Status**: 📝 Draft — @FEATURE decomposition pending
**Last Updated**: 2026-03-05

---

## Business Function

Provide a consistent dashboard structure across two access levels — **Branch** (all positions within the branch) and **Supervisor** (AM and above). Both levels share the same dashboard layout pattern: a home page with summary widgets, a collection list, and a performance summary. Content and scope differ by level.

## Why It Exists (First Principles)

- **Management by Exception**: Both branch staff and AMs need surfaced priorities — who is behind, what needs intervention — scoped to their level of accountability.
- **Area-Level Accountability**: AMs oversee multiple branches and need cross-branch performance, DPD movement, and their own escalated contract list in one place.
- **Staff Motivation**: Transparent personal metrics and gamified rankings drive healthy competition at the branch level.
- **Accountability Chain**: Branch → Area → Region aggregation enables consistent performance reporting upward.

---

## Feature Inventory

| Feature | Level | Status | Description |
|---------|-------|--------|-------------|
| Branch Home Dashboard | Branch | Draft | งานที่ต้องจัดการ widget + team performance summary + exception alerts |
| Branch Collection List | Branch | Draft | Priority-based work queue (same as Work Queue capability) accessible from dashboard |
| Branch Performance Summary | Branch | Draft | Team-level metrics: daily scorecard, contact compliance, active playbooks, leaderboard |
| Staff Self-Service Metrics | Branch (CO) | Draft | Personal metrics: tasks completed, PTP rate, visit success, SLA compliance |
| Supervisor Home Dashboard | Supervisor (AM+) | Draft | งานที่พื้นที่ต้องจัดการ widget + branch performance tracking (daily & weekly) + area metrics |
| AM's Responsible Contracts | Supervisor (AM+) | Draft | สัญญาที่อยู่ภายใต้การดูแลของพื้นที่ — escalated or manually added contracts; AM assigns to branch or handles directly |
| Branch Collection List (AM View) | Supervisor (AM+) | Draft | การติดตามหนี้ในแต่ละสาขา — full contract list per branch; AM can pull contracts into AM's responsible list |
| Area Performance Summary | Supervisor (AM+) | Draft | ยอดสินเชื่อ / ยอดการขาย daily and monthly vs. target; DPD movement (C to X, X to 30) |

---

## Dashboard Structure (Both Levels Share Same Pattern)

| Section | Branch Level | Supervisor Level (AM+) |
|---------|-------------|------------------------|
| **Summary Widget** | งานที่ต้องจัดการ — count of active tasks in CO's queue | งานที่พื้นที่ต้องจัดการ — count of contracts in AM's responsible list |
| **Work Setting** | การเรียงลำดับงาน / Strategy ในการทำงาน | การเรียงลำดับงาน / Strategy ในการทำงาน |
| **Tracking Table** | Team workload per CO (queue size, completed, rate, PTP, alerts) | Branch performance per branch (ติดตามหนี้ + เสนอขาย metrics — วันนี้ and สัปดาห์นี้) |
| **Collection List** | Priority-based contract queue (Work Queue) | สัญญาที่อยู่ภายใต้การดูแลของพื้นที่ + การติดตามหนี้ในแต่ะสาขา |
| **Performance Summary** | Daily / weekly / monthly scorecard + contact compliance | Daily and monthly area targets + DPD movement |

---

## Business Rules

---

### A. Branch Level (All Branch Positions)

#### A1. Branch Home Dashboard

| Component | Description |
|-----------|-------------|
| **งานที่ต้องจัดการ** | Summary widget: total active tasks in queue today; click navigates to Work Queue |
| **การตั้งค่าการทำงาน** | Work configuration: การเรียงลำดับงาน (sort order), Strategy ในการทำงาน |
| **ติดตามผลการทำงานของทีม** | Per-CO row: queue size, completed, completion rate, PTP amount, exception flags |
| **Exception Alerts** | Surfaced for supervisor role within branch (see alert types below) |
| **Contact Compliance** | Real-time compliance status for the team (100% = no violations) |

#### A2. Branch Exception Alert Types

| Alert | Trigger | Action |
|-------|---------|--------|
| Idle Staff | No activity > 1 hour during work hours | Review, send message |
| Low Performance | Completion rate < 40% at midday | Intervene, reassign |
| Contact Blocked | Customer reached daily contact limit | Acknowledge, plan next-day |
| Playbook Stuck | Step awaiting supervisor approval | Approve / reject |
| SLA Breach | Task overdue by > 4 hours | Reassign or escalate |

#### A3. Branch Performance Summary

| Component | Purpose |
|-----------|---------|
| Daily Scorecard | Team-level metrics: today / this week / this month / vs. target |
| Active Playbooks Panel | Per-playbook: case count, on-track / at-risk / failed / succeeded |
| Staff Self-Service Metrics | Personal: tasks completed, PTP rate, visit success, SLA compliance |
| Monthly Objectives Tracker | Count of succeeded / in-progress / failed objectives per CO |
| Branch Rank & Leaderboard | Gamified ranking within branch (🥇🥈🥉); composite score = completion rate + PTP rate + SLA compliance + contact compliance |

---

### B. Supervisor Level (AM and Above)

#### B1. Supervisor Home Dashboard (หน้าหลัก)

| Component | Description |
|-----------|-------------|
| **งานที่พื้นที่ต้องจัดการ** | Count of contracts in สัญญาที่อยู่ภายใต้การดูแลของพื้นที่; click navigates to AM's contract list |
| **การตั้งค่าการทำงาน** | Work configuration: การเรียงลำดับงาน (sort order), Strategy ในการทำงาน |
| **ติดตามผลการทำงานของสาขา** | Branch performance table — วันนี้ and สัปดาห์นี้; ติดตามหนี้ and เสนอขาย metrics per branch under AM's area |
| **ภาพรวมการทำงานประจำวันนี้** | Daily: ยอดสินเชื่อ and ยอดการขาย (ส่วนต่าง / ทำได้ / เป้าหมาย) |
| **ภาพรวมการทำงานเดือนนี้** | Monthly cumulative vs. target for same metrics |
| **การไหลของ DPD** | DPD movement rates (C to X, X to 30) for current and prior month |

#### B2. สัญญาที่อยู่ภายใต้การดูแลของพื้นที่ (AM's Responsible Contracts)

Contracts enter this list via two routes:
1. **Auto-escalated** — "ส่งเรื่องให้ผู้จัดการพื้นที่" outcome on เอาวันนัดชำระ expiry. **No new task is created.** Contract moves here for AM to manage.
2. **Manually added** — AM pulls a contract from การติดตามหนี้ในแต่ละสาขา.

AM actions per contract: assign back to original branch, reassign to another branch in area, or handle directly.

| Column | Description |
|--------|-------------|
| ชื่อ-นามสกุล (ชื่อเล่น) | Customer full name and nickname |
| Due date | Relevant due date (color-coded: overdue = orange/red) |
| สถานะการจ่าย | Payment status badge |
| ยอดตามคาดการณ์ | Forecasted payment amount |
| วันที่ติดต่อล่าสุด | Date of most recent contact |
| ผลการติดต่อล่าสุด | Outcome of most recent contact |
| สาขาต้นทาง | Source branch |
| มอบหมายให้สาขา | Dropdown: assign to branch (or keep with AM) |
| หมายเหตุ | Free-text note |

#### B3. การติดตามหนี้ในแต่ละสาขา (Branch Collection List — AM View)

AM reads each branch's full contract list and can pull contracts into AM's responsible list. Clicking a row opens the customer page (same drill-through as Work Queue).

**Filters**: Search by ชื่อ-นามสกุล / เลขที่สัญญา / เบอร์โทร / เลขโปรเจคติด / เลขบัตรประชาชน; filter by เลขแมนเอดิต, ถ่วตัวรอง.

| Column | Description |
|--------|-------------|
| ความเสี่ยง | Risk level (color-coded: เสี่ยงสูง red, เสี่ยงกลาง orange, เสี่ยงต่ำ green/blue) |
| ชื่อ-นามสกุล (ชื่อเล่น) | Customer full name and nickname |
| % ต่อพอร์ต | Contract weight as % of branch portfolio |
| Due date | Relevant due date |
| Action | Recommended action for current Objective |
| ผลลัพธ์ที่คาดหวัง | Current Objective (e.g., เอาวันนัดชำระ) |
| สถานการจ่าย | Payment status badge |
| ยอดตามคาดการณ์ | Forecasted payment amount |
| วันที่ติดต่อล่าสุด | Date of most recent contact |
| ผลการติดตามล่าสุด | Outcome of most recent contact |
| สาขา | Branch name |
| ผู้รับผิดชอบเพิ่มเติม | Additional CO(s) assigned |
| เรื่องกฎสัญญา | Compliance flag (⚠ if issue exists) |

---

## NFRs

| NFR | Requirement |
|-----|-------------|
| Real-time updates | Both dashboards update without page reload (near-real-time, ≤ 30 seconds) |
| Exception alerting | Branch exceptions surfaced within 5 minutes of trigger condition |
| Historical data | Monthly objectives and PTP data retained for at least 12 months |
| AM escalation atomicity | When "ส่งเรื่องให้ผู้จัดการพื้นที่" fires, contract must appear in AM's list atomically — no task gap, no duplicate |
| Branch list performance | การติดตามหนี้ในแต่ละสาขา must render within 2 seconds for full branch portfolio |
