# Feature: Pre-Approval Detail View

**Capability**: Restructure Pre-Approval — [CAPABILITY](../CAPABILITY.md)
**Product**: Onigiri — [PRODUCT](../../../PRODUCT.md)
**Engineering Owner**: TBD
**Status**: Spec

---

## User Story

As a **CA (Credit Analyst)**, I want to see the full restructuring plan proposed by the sales team — including the existing contract details, the proposed new terms, the projected payment schedule, and any notes from the sales officer — so that I have everything I need to make an informed pre-approval decision without requesting additional information.

## Job-to-be-Done

Give CA a complete, self-contained view of the restructuring proposal. CA must not need to contact the sales team or look up the contract separately before deciding.

---

## Acceptance Criteria

1. The Pre-Approval Detail View is accessible by clicking a row in the Pre-Approval Worklist Modal. It opens as a full page (not an inline expansion).
2. The page is accessible only to authenticated users with `CA` group_role. Any other role receives HTTP 403.
3. The page is read-only for all sections except the CA decision panel (see FEATURE_pre-approval-decision.md).
4. If the pre-approval record is no longer in `PENDING_CA_REVIEW` status when the page is loaded (already decided), the decision panel is disabled and a banner shows the current status and decision details.

---

## Page Structure

### Section 1 — Pre-Approval Header

| Field | Source |
|-------|--------|
| Pre-Approval Reference | Pre-approval record |
| Status | Pre-approval record (`PENDING_CA_REVIEW`) |
| Submitted By | Sales officer name and branch |
| Submitted Date | Pre-approval record |
| Days Pending | Calculated |

---

### Section 2 — Existing Contract Summary *(snapshotted from Core Banking at plan creation)*

| Field | Source |
|-------|--------|
| Contract Number | Snapshot |
| Customer Name | Snapshot |
| Product Type | Snapshot |
| Outstanding Balance | Snapshot |
| Current Monthly Instalment | Snapshot |
| Current Remaining Tenure (months) | Snapshot |
| Current Interest Rate | Snapshot |
| Contract Status at Snapshot Time | Snapshot |
| Snapshot Captured At | Timestamp |

---

### Section 3 — Proposed Restructuring Terms

Side-by-side comparison of current vs. proposed:

| Field | Current (from snapshot) | Proposed (by sales) |
|-------|------------------------|---------------------|
| Monthly Instalment | | |
| Tenure (months) | | |
| Interest Rate | | |
| Restructuring Fee | N/A | |

---

### Section 4 — Proposed Payment Schedule

Month-by-month table of the proposed instalment plan:

| Month | Payment Date | Instalment Amount | Principal Component | Interest Component | Outstanding Balance |
|-------|-------------|------------------|--------------------|--------------------|---------------------|
| 1 | … | … | … | … | … |
| … | | | | | |

Displayed as a paginated table (first 12 months visible; expandable to full schedule).

---

### Section 5 — Sales Officer Notes

Free-text notes submitted by the sales officer at plan creation time. Displayed as-is. Empty section hidden if no notes were provided.

---

### Section 6 — Decision Panel

See [FEATURE_pre-approval-decision.md](FEATURE_pre-approval-decision.md) for the full decision panel spec.

Rendered at the bottom of the page. Contains the three CA actions: Approve, Change, Reject.

---

## Edge Cases and Error States

| Condition | Expected Behavior |
|-----------|-------------------|
| Pre-approval already decided when page loads | Decision panel disabled; status banner shown with decision and actor details |
| Contract snapshot is incomplete (Core Banking returned partial data at creation time) | Affected fields display "—" with a warning note: "Data unavailable at snapshot time." Decision panel remains active — CA can still decide. |
| Session expires while CA is on the detail page | On re-authentication, page reloads with current state; if already decided by another CA, banner shown |

---

## Out of Scope

- Live re-fetch of contract data from Core Banking (snapshot only)
- Customer contact details (sales officer manages customer communication)
- Document upload or attachment on the pre-approval record

---

## Dependencies

- Pre-approval record must contain a complete snapshot from the Plan Builder feature.
- CA group_role authentication.
- Pre-Approval Decision feature provides the decision panel rendered in Section 6.
