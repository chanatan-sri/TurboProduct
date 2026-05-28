# Feature: Pre-Approval Request Tracker

**Capability**: Restructure Pre-Approval — [CAPABILITY](../CAPABILITY.md)
**Product**: Onigiri — [PRODUCT](../../../PRODUCT.md)
**Engineering Owner**: TBD
**Status**: Spec

---

## User Story

As a **sales officer**, I want to see the current status of all restructuring pre-approval requests I have submitted, including CA's decision or revised terms, so that I can follow up with the customer as soon as a result is available.

## Job-to-be-Done

Close the feedback loop between submission and customer conversation. Once a plan is sent to CA, the sales officer needs a single page to monitor all outstanding requests — without needing to ask CA directly or chase through a separate channel.

---

## Acceptance Criteria

### List View

1. The Pre-Approval Request Tracker is accessible to sales officers from their main navigation. It shows all pre-approval records created by the logged-in user's branch or team (scoping TBD — see Open Questions).
2. Each row in the list displays:

| Column | Source | Notes |
|--------|--------|-------|
| Pre-Approval Reference | System-generated ID | Unique identifier |
| Contract Number | Pre-approval record (snapshotted) | |
| Customer Name | Pre-approval record (snapshotted from Core Banking) | |
| Proposed Tenure | Pre-approval record | Months |
| Proposed Monthly Instalment | Pre-approval record | Currency formatted |
| Status | Pre-approval record | Badge: Pending / Approved / Pending Revision / Revision Confirmed / Revision Rejected / Rejected / Expired |
| Submitted Date | Pre-approval record | Date submitted for CA review |
| Last Updated | Pre-approval record | Date of most recent status change |

3. The list is sorted by **Last Updated descending** (most recently updated first) by default.
4. Filters available: Status (multi-select), Submitted Date range.
5. Clicking a row opens the pre-approval detail — a read-only view for the sales officer showing the same plan data as the CA detail view, plus CA's decision, comments, and revised terms if applicable.

### Status Notifications

6. When a pre-approval record transitions to `APPROVED`, `REJECTED`, or `PENDING_REVISION`, the originating sales officer is notified (mechanism TBD — in-app notification or Sensei task — see CAPABILITY.md Open Question #4 for mechanism alignment).
7. The tracker list highlights rows that have changed status since the sales officer last viewed them (unread indicator).

### Revision Confirmation / Rejection Flow

8. When a record is in `PENDING_REVISION` status, the sales officer can open the detail and see CA's revised terms side-by-side with the original proposed terms, along with CA's mandatory notes explaining the reason for the change.
9. Two actions are available on the detail view when status is `PENDING_REVISION`:
   - **Confirm Revision** — sales officer accepts CA's revised terms after confirming with the customer.
   - **Reject Revision** — sales officer declines CA's revised terms. Requires a brief note (optional, max 200 characters).
10. On **Confirm**: pre-approval status transitions to `REVISION_CONFIRMED`. The revised terms become the active plan. The record is now eligible to confer risk-reduction benefit on a linked application.
11. On **Reject**: pre-approval status transitions to `REVISION_REJECTED`. The record is closed. The contract is freed — a new pre-approval for the same contract may now be submitted.
12. After either action, the tracker row updates its status badge accordingly.

---

## Edge Cases and Error States

| Condition | Expected Behavior |
|-----------|-------------------|
| Pre-approval already linked to a submitted application | Row displays a "Linked to application [ref]" tag. Revision Confirm and Reject are disabled — plan is already in use. |
| Sales attempts to submit a new plan while the contract already has an active pre-approval | Blocked at the Plan Builder step with message: "This contract already has an active pre-approval [reference]. A new plan can be submitted after the existing one is closed." |
| Pre-approval expired before sales officer confirms revision | Status displayed as `Expired`. Revision Confirm is disabled. Sales officer must build a new plan. |
| Pre-approval `APPROVED` but attached application has already been submitted | Status and linked application reference shown. No further sales action required. |

---

## Out of Scope

- Editing a plan after it has been submitted to CA (`PENDING_CA_REVIEW` or later statuses are read-only for the sales officer)
- Cancelling a pre-approval request after it has been submitted (requires supervisor action — out of scope)

---

## Dependencies

- Pre-Approval Decision feature must write CA's decision back to the pre-approval record before this tracker can reflect it.
- Notification mechanism (in-app or Sensei) — open question.
