# Feature: Pre-Approval Worklist Modal

**Capability**: Restructure Pre-Approval — [CAPABILITY](../CAPABILITY.md)
**Product**: Onigiri — [PRODUCT](../../../PRODUCT.md)
**Engineering Owner**: TBD
**Status**: Spec

---

## User Story

As a **CA (Credit Analyst)**, I want to access pending restructuring pre-approval requests from within my existing application approval worklist, via a clearly separated entry point, so that I can action pre-approvals without confusing them with normal loan applications.

## Job-to-be-Done

Surface a distinct, low-friction entry point on the CA worklist for a different type of work item. The pre-approval queue must be visually and functionally separate from the loan application queue — they serve different purposes and have different action sets.

---

## Acceptance Criteria

### Trigger Button

1. A **"Pre-Approval Requests"** button is displayed on the CA Application Approval worklist page, visually distinct from the application list (e.g., in the page header or action bar area).
2. The button displays a **badge count** showing the number of pre-approval requests currently in `PENDING_CA_REVIEW` status and routed to `CA` group_role. Badge updates in near-real-time (≤ 30s).
3. The button is only visible to users with `CA` group_role. Users with other group roles do not see it.

### Modal Content

4. Clicking the button opens a modal overlay. The modal does not navigate away from the worklist page.
5. The modal header displays: "Pre-Approval Requests" and the current pending count.
6. The modal lists all pre-approval records with status `PENDING_CA_REVIEW` routed to `CA` group_role. Each row displays:

| Column | Source |
|--------|--------|
| Contract Number | Pre-approval record (snapshot from Core Banking) |
| Full Name | Pre-approval record (customer full name, snapshot from Core Banking) |
| Branch Name | Pre-approval record (branch of the submitting sales officer) |
| Request Date/Time | Pre-approval record (timestamp when submitted for CA review) |

7. The modal list is sorted by Request Date/Time ascending (oldest first).
8. A **search bar** is displayed at the top of the modal. It filters the list in real-time as CA types. Search matches against Contract Number (exact or prefix) and Full Name (contains match). Search is case-insensitive.
8. Clicking a row in the modal navigates to the Pre-Approval Detail View for that request. The modal closes on navigation.
9. A pre-approval record that exits `PENDING_CA_REVIEW` (decided by CA) is removed from the modal list within 30 seconds.
10. If there are no pending pre-approval requests, the modal displays an empty state: "No pre-approval requests pending your review."
11. The modal is closeable via an explicit close button or by clicking outside the modal. Closing returns the user to the worklist.

---

## Separation from Application List

The pre-approval modal is intentionally **not integrated** into the main application list. Reasons:
- Pre-approval requests are not workflow applications — they have no `state` in the underwriting state machine.
- The action set is different: Approve / Change / Reject vs. the application's Approve / Reject / Request Document Upload.
- Mixing them would create sorting and filtering complexity with no benefit.

---

## Edge Cases and Error States

| Condition | Expected Behavior |
|-----------|-------------------|
| A pre-approval is actioned by CA while the modal is open | The row disappears on next modal refresh (≤ 30s); any decision submitted against an already-decided record returns a conflict error |
| User's session times out while modal is open | On next interaction, modal re-fetches; stale list is replaced |

---

## Out of Scope

- Bulk CA decisions on multiple pre-approvals at once
- Advanced filtering (by branch, date range) — search by contract number and full name is sufficient for this view

---

## Dependencies

- CA group_role authentication (same mechanism as Application Approval worklist).
- Pre-approval records must have `status = PENDING_CA_REVIEW` and `required_ca_role = CA` written by the Plan Builder on submission.
