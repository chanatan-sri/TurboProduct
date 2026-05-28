# CHANGELOG_019 — Restructure Pre-Approval Capability

**Date**: 2026-05-25
**Author**: chanatan.sri@turbo.co.th
**Layer Affected**: Capability (new) + Product (registry update)
**Branch**: loan-origination-with-cash

---

## What Changed

### New: Restructure Pre-Approval Capability

Added a new capability at `capabilities/restructure-pre-approval/` with five features.

**Capability**

- `CAPABILITY.md` — defines the Restructure Pre-Approval capability: plan builder, CA pre-approval flow, risk level reduction mechanism, lifecycle states, and integration with RAE.

**Features**

| Feature | File | Purpose |
|---------|------|---------|
| Restructure Plan Builder | `FEATURE_restructure-plan-builder.md` | Sales team selects active Core Banking contract, enters proposed restructuring terms, previews payment schedule, and chooses to submit for CA pre-approval or proceed directly to application input. |
| Pre-Approval Request Tracker | `FEATURE_pre-approval-request-tracker.md` | Sales-side list page showing all pre-approval requests and their current status. Includes revision confirmation flow when CA changes plan terms. |
| Pre-Approval Worklist Modal | `FEATURE_pre-approval-worklist-modal.md` | Button + modal on the CA Application Approval worklist, separate from the application list. Scoped to `CA` group_role. Badge count reflects pending requests. |
| Pre-Approval Detail View | `FEATURE_pre-approval-detail-view.md` | CA-facing full page showing contract snapshot, current vs. proposed terms side-by-side, payment schedule, and sales officer notes. |
| Pre-Approval Decision | `FEATURE_pre-approval-decision.md` | CA's three actions: Approve (status → APPROVED), Change (CA edits terms → PENDING_REVISION → sales re-confirms → REVISION_CONFIRMED), Reject (status → REJECTED, reason required). |

**Product Registry Update**

- `PRODUCT.md` — Restructure Pre-Approval added to the Capability Registry.

---

## Architecture Decision: RAE-Driven Risk Reduction

The pre-approval does not bypass the underwriting workflow. Instead, a CA-approved pre-approval plan is attached to the application as `pre_approval` object in the application JSON. The RAE restructuring strategy includes a policy that evaluates this object. A valid, non-expired `APPROVED` or `REVISION_CONFIRMED` pre-approval produces **risk level 10–20**, routing the application to `SALE_BRANCH` or `SALE_AREA` for final approval.

**Why this approach over workflow bypass:**
- Topology remains unchanged — no special-case state machine paths
- The RAE trace provides a full audit of why the risk level was reduced
- The pre-approval's impact is configurable (the RAE rule can be tuned or deactivated without code change)
- Consistent with the zero-code rule change model already established in the Risk Assessment Engine

---

## Pre-Approval Lifecycle

```
DRAFT → PENDING_CA_REVIEW → APPROVED
                           → REJECTED
                           → PENDING_REVISION → REVISION_CONFIRMED
```

`APPROVED` and `REVISION_CONFIRMED` both confer the risk-reduction benefit on a linked application (if not expired).

---

## Key Design Decisions

| Decision | Rationale |
|----------|-----------|
| Pre-approval is optional | Forcing pre-approval adds friction to simple cases. Sales team decides based on their read of the customer and the case complexity. |
| CA-only routing for pre-approvals | Pre-approval is a credit judgment on restructuring terms, not a branch-level decision. CA is always the pre-approval authority regardless of the expected downstream risk level. |
| Separation from application worklist | Pre-approval requests are not workflow applications. Mixing them with `pending_approval` applications would require different columns, filters, and actions in the same table — creating UX and maintenance complexity. |
| Contract data snapshot | Core Banking contract state at plan creation time is the basis for CA's review. Re-fetching live data at review time could surface changes that invalidate the plan without the sales officer's knowledge. |
| Sales must confirm CA revision | CA changing terms creates a new plan that the customer has not yet agreed to. The sales officer re-confirms with the customer before the plan is treated as accepted. |

---

## Open Questions (5)

1. Pre-approval expiry window (recommend 30 days from approval date)
2. Whether sales can reject CA's revision and re-submit a new plan
3. Multiple pre-approvals per contract — which one is linked to the application
4. Core Banking contract lookup API/mechanism (read-only; Onigiri does not write at pre-approval time)
5. Risk level 10–20 for pre-approved applications — fixed value or RAE-strategy-dependent

---

## Documents Created / Modified

| File | Change Type |
|------|------------|
| `capabilities/restructure-pre-approval/CAPABILITY.md` | Created |
| `capabilities/restructure-pre-approval/features/FEATURE_restructure-plan-builder.md` | Created |
| `capabilities/restructure-pre-approval/features/FEATURE_pre-approval-request-tracker.md` | Created |
| `capabilities/restructure-pre-approval/features/FEATURE_pre-approval-worklist-modal.md` | Created |
| `capabilities/restructure-pre-approval/features/FEATURE_pre-approval-detail-view.md` | Created |
| `capabilities/restructure-pre-approval/features/FEATURE_pre-approval-decision.md` | Created |
| `PRODUCT.md` | Updated — capability registry |
