# CHANGELOG_016 — Application Approval Capability

**Date**: 2026-05-25
**Author**: chanatan.sri@turbo.co.th
**Layer Affected**: Capability (new) + Product (registry update)
**Branch**: loan-origination-with-cash

---

## What Changed

### New: Application Approval Capability

Added a new capability at `capabilities/application-approval/` with four features.

**Capability**

- `CAPABILITY.md` — defines the Application Approval capability: approval routing, worklist, role-gated detail view, and decision actions.

**Features**

| Feature | File | Purpose |
|---------|------|---------|
| Approval Routing Assignment | `FEATURE_approval-routing-assignment.md` | On `pending_approval` entry: reads RAE `risk_level`, writes `required_approver_role` to application record. Auto-declines risk level 70 and 99 (bypasses `pending_approval`). Strict role matching — no upward delegation. |
| Approver Worklist | `FEATURE_approver-worklist.md` | `/worklist` page scoped to the authenticated user's exact role. Shows application reference, borrower name, product, loan amount, risk level, days pending. Sorted oldest-first. Real-time removal on state exit (≤ 30s). |
| Application Detail View | `FEATURE_application-detail-view.md` | Full application detail page with 7 visibility tiers. Tier 1–2 (all roles): basic info + Smart Form fields. Tier 3 (AM+): financials, DTI, LTV. Tier 4–6 (CA+): NCB bureau result, RAE evaluation trace, collateral valuation. Tier 7 (CA Manager+): full workflow audit trail. Visibility enforced server-side. |
| Approval Decision | `FEATURE_approval-decision.md` | Three actions from the detail page: Approve (→ `create_facility`, writes `approval_snapshot`), Reject (→ `rejected`, requires reason), Request Document Upload (→ `draft`, requires doc type). Optimistic lock prevents double-decision. All decisions written to immutable audit log. |

**Product Registry Update**

- `PRODUCT.md` — Application Approval added to the Capability Registry.

---

## Rationale

The `pending_approval` state existed in the Underwriting Workflow topology since the initial Atlas document, but the capability responsible for its content — routing logic, approver UI, decision actions — was never specified. This changelog formalizes that specification.

The decision to own the worklist inside Onigiri (rather than routing through Sensei) reflects that credit approval is a credit-department function (CA, AM, CRO) with a distinct audience and lifecycle from branch operational tasks.

---

## Key Decisions and Alternatives Rejected

| Decision | Alternative Considered | Reason Rejected |
|----------|----------------------|-----------------|
| Strict role matching (no upward delegation) | Allow CA Manager to approve CA-level applications | Upward delegation creates audit ambiguity — it is unclear whether a CA Manager's approval is covering for absence or overriding a CA-level decision. Strict matching forces explicit escalation processes to be defined and logged. |
| Worklist in Onigiri | Route `TaskCreationRequest` to Sensei for approver queue | Sensei's audience is branch operational staff. Credit approvers (CA, CRO, CEO) may be centralized. Mixing the two queues adds operational complexity with no reuse benefit. |
| 7 visibility tiers on the detail view | Single view for all roles | A single view would either under-serve senior approvers (missing confidential data) or over-expose sensitive bureau and risk trace data to branch staff. Tiered visibility is necessary. |
| Optimistic lock for concurrent decisions | Last-write-wins | Two approvers approving the same application would create two audit log entries for a single decision. Optimistic lock ensures exactly one decision is committed. |

---

## Open Questions Logged

1. Supervisor role reassignment (when designated approver is unavailable)
2. Approver notification mechanism on `pending_approval` entry (in-app vs. Sensei task)
3. SLA / aging policy for applications sitting in `pending_approval` beyond a threshold
4. CO / SCO / BM / SBM visibility of Tier 3 financials (confirm inclusion or exclusion)

---

## Documents Modified

| File | Change Type |
|------|------------|
| `capabilities/application-approval/CAPABILITY.md` | Created |
| `capabilities/application-approval/features/FEATURE_approval-routing-assignment.md` | Created |
| `capabilities/application-approval/features/FEATURE_approver-worklist.md` | Created |
| `capabilities/application-approval/features/FEATURE_application-detail-view.md` | Created |
| `capabilities/application-approval/features/FEATURE_approval-decision.md` | Created |
| `PRODUCT.md` | Updated — capability registry |
