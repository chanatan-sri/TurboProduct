# Feature: Approver Worklist

**Capability**: Application Approval — [CAPABILITY](../CAPABILITY.md)
**Product**: Onigiri — [PRODUCT](../../../PRODUCT.md)
**Engineering Owner**: TBD
**Status**: Spec

---

## User Story

As a **credit approver** (CA, AM, CRO, or equivalent role), I want a worklist that shows only the loan applications assigned to me and awaiting my approval decision, so that I can identify which applications need my action without filtering through irrelevant queues.

## Job-to-be-Done

Give approvers a focused, role-scoped queue. The worklist is not a general application search — it is the operational surface that tells an approver exactly what they are accountable for right now.

---

## Acceptance Criteria

1. The worklist page (`/worklist`) is accessible only to authenticated users with a recognized approver role. Unauthenticated or non-approver roles receive HTTP 403.
2. The worklist renders applications that satisfy **both** of the following conditions:
   - Assigned to the logged-in user by the external **Worklist Distribution System** **AND**
   - `state = pending_approval` with `required_approver_role` exactly matching the authenticated user's `group_role` (Onigiri's secondary role-boundary enforcement — no upward delegation).
   
   If the distribution system assigns an application whose `required_approver_role` does not match the user's `group_role`, Onigiri silently excludes it from the rendered list.
3. Each row in the worklist displays the following columns:

| Column | Source | Notes |
|--------|--------|-------|
| Application Reference | Application record | Unique identifier |
| Borrower Name | Smart Form (borrower section) | Full name |
| Product / Campaign | Campaign configuration | Product type and campaign name |
| Requested Loan Amount | Smart Form (loan setup) | Currency formatted |
| Risk Level | RAE output on application record | Numeric value (10 / 20 / 30 / 40 / 50 / 60) |
| Days Pending | Calculated from `pending_approval` entry timestamp to now | Integer — days since entered this state |
| Date Submitted | `pending_approval` entry timestamp | Date only (no time) |

4. The worklist is sorted by **Days Pending descending** (oldest applications first) by default. The user may re-sort by any column.
5. The worklist reflects applications that have entered `pending_approval` within the last **90 days**. Applications older than 90 days in this state are surfaced with a visual age warning (pending SLA definition — see Open Questions in CAPABILITY.md).
6. An application that exits `pending_approval` (approved, rejected, returned to draft) is **removed from the worklist** within 30 seconds.
7. The worklist includes a count of total pending items visible to the logged-in user, displayed in the page header.
8. Clicking a row navigates to the Application Detail View for that application.
9. An empty worklist state (no applications matching the user's role) displays a "No applications pending your review" message — not an error.

---

## Filtering and Search

| Filter | Options | Behaviour |
|--------|---------|-----------|
| Risk Level | Multi-select: 10, 20, 30, 40, 50, 60 | Narrows to selected risk levels only |
| Days Pending | Range: e.g., "0–7 days", "8–30 days", "> 30 days" | Narrows by age bracket |
| Product Type | Multi-select from available campaign names | Narrows by product |
| Application Reference | Free text search | Exact or prefix match |

All filters are additive (AND logic). Default state: no filters applied (show all matching applications).

---

## Edge Cases and Error States

| Condition | Expected Behavior |
|-----------|-------------------|
| Authenticated user has no approver role | HTTP 403 — not authorized to view worklist |
| Authenticated user has an approver role but no applications match | Empty state message — not an error |
| Application transitions out of `pending_approval` while displayed in worklist | Row removed on next refresh (≤ 30s); no stale action is possible (server validates state at decision time) |
| Two approvers with the same role view the same application | Both see it in their worklist — only the first to submit a decision will succeed; the second receives a conflict error (see Approval Decision feature) |

---

## Out of Scope

- Cross-role worklist view (a manager seeing another role's queue)
- Bulk approval actions
- Worklist customization (column reordering, saved filters)

---

## Dependencies

- Approval Routing Assignment feature must have written `required_approver_role` before the application appears here.
- Authentication system must expose the authenticated user's `group_role` as a verifiable claim (not client-supplied).
- **Worklist Distribution System** must provide Onigiri with the assignment list for the logged-in user. Integration contract (API, event, or query mechanism) is a pending dependency — see Open Question #1 in CAPABILITY.md.
