# Capability: Performance Dashboard

**Product**: Sensei — [PRODUCT](../../PRODUCT.md)
**Portfolio**: Operations
**Product Owner**: TBD (Operations PO)
**Status**: 📝 Draft — @FEATURE decomposition pending
**Last Updated**: 2026-03-04

---

## Business Function

Provide real-time visibility into team operations for supervisors and self-service performance tracking for individual field staff — enabling management by exception, staff accountability, and gamified motivation in high-volume branch operations.

## Why It Exists (First Principles)

- **Management by Exception**: Supervisors managing 10–15 staff and 3,000+ tasks cannot review everything. They need to see exceptions — who is behind, which playbooks are failing, what needs intervention.
- **Staff Motivation**: Transparency in personal performance metrics and gamified rankings (leaderboard) drive healthy competition and self-improvement.
- **Accountability Chain**: Branch → Area → Region aggregation enables consistent performance reporting upward.

---

## Feature Inventory

| Feature | Status | Description |
|---------|--------|-------------|
| Team Workload Table | Draft | Per-staff row: queue size, completed, completion rate, PTP amount, alert flags |
| Active Playbooks Panel | Draft | Per-playbook: case count, on-track/at-risk/failed/succeeded, progress bar |
| Exception Panel | Draft | Supervisor alert queue: 5 alert types requiring action |
| Daily Scorecard | Draft | Team-level aggregated metrics: today / this week / this month / vs. target |
| Contact Compliance Status | Draft | Real-time compliance status for the team (100% = no violations) |
| Staff Self-Service Metrics | Draft | Personal metrics: tasks completed, PTP rate, visit success, SLA compliance |
| Monthly Objectives Tracker | Draft | Count of succeeded/in-progress/failed objectives from assigned playbooks |
| Branch Rank & Leaderboard | Draft | Gamified ranking within branch with medal indicators (🥇🥈🥉) |
| Supervisor Feedback | Draft | Messages and coaching notes from supervisor to individual staff |

---

## Business Rules

### Exception Alert Types

| Alert | Trigger | Supervisor Action |
|-------|---------|------------------|
| Idle Staff | No activity for > 1 hour during work hours | Review, send message |
| Low Performance | Completion rate < 40% at midday | Intervene, reassign |
| Contact Blocked | Customer reached daily contact limit | Acknowledge, plan next-day |
| Playbook Stuck | Playbook step awaiting supervisor approval | Approve / reject |
| SLA Breach | Task overdue by > 4 hours | Reassign or escalate |

### Staff Self-Service Components

| Component | Purpose |
|-----------|---------|
| Metrics Dashboard | Personal progress bars: tasks completed, PTP rate, visit success, SLA compliance |
| Monthly Objectives | Count of succeeded/in-progress/failed objectives from assigned playbooks |
| Branch Rank | Gamified ranking within branch with medal indicators |
| Team Leaderboard | Ranked list of team members with task count and PTP rate |
| Supervisor Feedback | Messages and coaching notes from supervisor |

### Leaderboard Design Rules

- Leaderboard ranks all COs in the branch by composite score
- Composite score includes: task completion rate, PTP rate, SLA compliance, contact compliance
- Medal indicators: 🥇 Top performer, 🥈 Second, 🥉 Third
- Supervisor can add coaching notes visible only to the individual CO

---

## NFRs

| NFR | Requirement |
|-----|-------------|
| Real-time updates | Supervisor dashboard updates without page reload (near-real-time, ≤ 30 seconds) |
| Exception alerting | Exceptions surfaced in supervisor panel within 5 minutes of trigger condition |
| Historical data | Monthly objectives and historical PTP data retained for at least 12 months |
