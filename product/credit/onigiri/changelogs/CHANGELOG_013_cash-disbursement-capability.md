# CHANGELOG_013: Cash Disbursement Capability

**Date**: 2026-03-31
**Layer Affected**: Capability (new), Product (registry update)
**Author**: Product session — @CAPABILITY mode

---

## What Changed

### New Capability

**`product/credit/onigiri/capabilities/cash-disbursement/CAPABILITY.md`** — created

Defines the **Cash Disbursement** capability as the cash-path counterpart to the existing Disbursement Orchestration capability (non-cash path). This capability owns all states from `NeedConfCash` through `Funded` in the cash disbursement flow:

- `NeedConfCash` — variance gate (system decision)
- `ConfirmationCash` — officer confirmation (human action, cash-specific rules)
- `CreateFacilityCash` — Core Banking Create Facility (system, skip-guard idempotency)
- `CreateLoanDisbCash` — Core Banking Create Loan + Disbursement (system, hard-block idempotency)
- `QA_cash` — post-disbursement compliance review (human action, post-facto audit)

**Why a separate capability (not folded into Disbursement Orchestration):**
The cash path inverts the QA / disbursal sequence. In the non-cash path, QA gates disbursement (funds can be withheld). In the cash path, funds are released before QA — making QA a post-facto audit, not a control gate. The business rules, actor responsibilities, and risk profile are sufficiently distinct to warrant a separate capability boundary with its own Product Owner accountability.

---

### New Feature Specs (3)

**`capabilities/cash-disbursement/features/FEATURE_cash-confirmation-gate.md`** — created (Concept)

Covers `NeedConfCash` and `ConfirmationCash` states. Cash-specific confirmation rules: variance trigger condition, officer confirm/reject actions, bypass logic. Explicitly distinct from non-cash confirmation (`waiting_for_confirmation` in Disbursement Orchestration). Blocked on: variance definition, actor role confirmation, SLA.

**`capabilities/cash-disbursement/features/FEATURE_cash-facility-loan-disbursement.md`** — created (Concept)

Covers `CreateFacilityCash` and `CreateLoanDisbCash` states. Documents the skip-guard (Create Facility) vs. hard-block (Create Loan + Disbursement) idempotency distinction. Hard-block is the primary defense against double-cash-disbursement in addition to the HWM channel lock. Blocked on: Core Banking API contracts (cash variant).

**`capabilities/cash-disbursement/features/FEATURE_post-disbursement-qa.md`** — created (Concept)

Covers `QA_cash` state. Documents post-disbursement nature: loan is active in Core Banking regardless of QA outcome. QA failure triggers document remediation, not loan cancellation. Blocked on: authoritative QA checklist from QA/Operations.

---

### Updated Files

**`product/credit/onigiri/PRODUCT.md`** — Capability Registry updated

Added `Cash Disbursement` row linking to the new CAPABILITY.md.

---

## Rationale

The cash disbursement path has been documented in PRODUCT.md state machine diagrams and ATLAS.md since initial product definition, but had no corresponding capability document. The non-cash path was formally specified in CHANGELOG_003 (Disbursement Orchestration). This changelog completes the capability coverage for the Decision phase by formally specifying the cash path as a first-class capability.

---

## Decision Log

| Decision | Rationale | Alternative Rejected |
|----------|-----------|----------------------|
| Cash Disbursement as a separate capability (not extension of Disbursement Orchestration) | Inverted QA/disbursal sequence, distinct actor model, and independent operational risk profile warrant separate PO accountability | Extending Disbursement Orchestration would conflate pre- and post-disbursement QA semantics, making the capability document ambiguous |
| Cash-specific confirmation rules (not shared with non-cash) | Cash confirmation evaluates variance pre-disbursement; non-cash confirmation evaluates loan amount change post-Matcha-callback. Different triggers, different actors, different remediation paths | Shared confirmation gate would require conditional branching that obscures which path owns which rule |

---

## Open Questions Inherited (Blocking Concept → Spec)

1. Variance definition for `NeedConfCash` gate — Risk/Product Owner
2. Cash confirmation actor (officer vs. supervisor) — Operations
3. Core Banking API contracts for cash Create Facility + Create Loan + Disbursement — Engineering/Architecture
4. Authoritative QA checklist — QA/Operations
5. `ConfirmationCash` timeout SLA and escalation — Operations
6. Recovery path for `CreateLoanDisbCash` Core Banking failure — Risk/Operations

---

## Follow-Up Actions Required

- [ ] Risk/Product Owner to define variance threshold for `NeedConfCash` gate
- [ ] Operations to confirm cash confirmation actor role
- [ ] Architecture to define Core Banking API contracts (cash disbursement variant)
- [ ] QA/Operations to confirm authoritative post-disbursement QA checklist
- [ ] ARCHITECTURE.md to be updated with cash disbursement Core Banking API specs when available
- [ ] Features to be advanced from `Concept` → `Spec` once blockers are resolved
