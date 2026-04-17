# Feature: Cash Facility & Loan Disbursement

**Parent Capability**: [Cash Disbursement](../CAPABILITY.md)
**Parent Product**: [Onigiri — Loan Origination System](../../../PRODUCT.md)
**Engineering Owner**: TBD
**Status**: Concept

---

## User Story

> As the **system**, I want to execute the Create Facility and Create Loan + Disbursement commands against Core Banking in sequence after cash confirmation, so that the loan facility is established and funds are physically disbursed to the borrower in a single, auditable, non-retryable operation.

---

## Job-to-be-Done

These two sequential, system-driven states represent the point of no return in the cash loan lifecycle. `CreateFacilityCash` establishes the loan facility record in Core Banking (reversible, idempotent). `CreateLoanDisbCash` triggers physical cash disbursement (irreversible, hard-block idempotent). Both must complete before QA can begin. Neither requires human action — they are execution steps driven by the state machine.

---

## States Covered

| State | Role | Reversibility |
|-------|------|---------------|
| `CreateFacilityCash` | System issues Create Facility API call to Core Banking | Idempotent (skip guard — safe to retry) |
| `CreateLoanDisbCash` | System issues Create Loan + Disbursement API call to Core Banking | **Irreversible** (hard block — must never re-execute) |

---

## Acceptance Criteria

### Create Facility (`CreateFacilityCash`)

- [ ] On entry to `CreateFacilityCash`, the system issues a Create Facility command to Core Banking with the loan application details.
- [ ] HWM is updated to `CreateFacilityCash` on entry, before the Core Banking call is made.
- [ ] **Skip guard (idempotency):** If a facility ID for this application already exists in Onigiri's record, the system skips the Core Banking call and uses the existing facility ID. No error is raised.
- [ ] The Core Banking request payload and response are stored in MongoDB before the state machine advances.
- [ ] On Core Banking success: application transitions to `CreateLoanDisbCash` automatically (system auto-advance, no human action).
- [ ] On Core Banking failure: the application remains in `CreateFacilityCash`. Alert fires. Manual intervention required. (Recovery path — Open Question 6.)
- [ ] The disbursement channel field remains read-only (HWM lock enforced since `Create Facility` HWM was reached).

### Create Loan + Disbursement (`CreateLoanDisbCash`)

- [ ] On entry to `CreateLoanDisbCash`, the system issues a Create Loan + Disbursement command to Core Banking using the facility ID from `CreateFacilityCash`.
- [ ] HWM is updated to `CreateLoanDisbCash` on entry, before the Core Banking call is made.
- [ ] **Hard block (idempotency):** If the `CreateLoanDisbCash` HWM has ever been set on this application, the execution step MUST NOT issue the Core Banking call under any circumstances — including retry, re-entry from Draft, or supervisor override. This is a non-negotiable safety control.
- [ ] The Core Banking request payload and response are stored in MongoDB before the state machine advances.
- [ ] On Core Banking success: application transitions to `QA_cash` automatically.
- [ ] On Core Banking failure: the application remains in `CreateLoanDisbCash`. Alert fires immediately. **No automatic retry.** Manual supervisor intervention required. (Recovery path — Open Question 6.)
- [ ] All financial fields are read-only after HWM reaches `CreateLoanDisbCash` (lockpoint group `ALL_FINANCIAL` per PRODUCT.md Cross-Capability Integrity Rules).

---

## Edge Cases & Error States

| Scenario | Expected Behavior |
|----------|-------------------|
| Application returned to Draft after `CreateFacilityCash` completes, then resubmitted | Skip guard fires on `CreateFacilityCash` — existing facility ID reused. No duplicate facility created. |
| Application returned to Draft after `CreateLoanDisbCash` completes, then resubmitted | Hard block fires on `CreateLoanDisbCash` — execution step is permanently skipped. Application cannot proceed through cash path again. Remediation: cancel and submit new application. |
| Core Banking returns an error on Create Facility | Application stays in `CreateFacilityCash`. Alert fires. No auto-retry. |
| Core Banking returns an error on Create Loan + Disbursement | Application stays in `CreateLoanDisbCash`. Alert fires. Supervisor must determine whether funds actually left. No auto-retry ever. |
| Core Banking timeout (no response within SLA) | Treat as failure. Alert fires. Manual investigation required to determine if funds were disbursed. |
| Duplicate Core Banking callback received | Ignored. Idempotency key already consumed. |

---

## Out of Scope

- Create Facility / Create Loan + Disbursement for the **non-cash path** — these are separate Core Banking commands issued by [Disbursement Orchestration](../../disbursement-orchestration/CAPABILITY.md) via `waiting_create_facility` and `waiting_create_loan_operation` states
- Core Banking API contract specification — defined in [ARCHITECTURE.md](../../../ARCHITECTURE.md)
- Fund transfer callbacks (non-cash) — owned by [Disbursement Orchestration](../../disbursement-orchestration/CAPABILITY.md)

---

## Dependencies

| Dependency | Type | Status |
|------------|------|--------|
| Core Banking API contract: Create Facility (cash) — endpoint, payload, response, error codes | Engineering (Architecture) | **Blocking** — required before Concept → Spec |
| Core Banking API contract: Create Loan + Disbursement (cash) — endpoint, payload, auth, timeout SLA | Engineering (Architecture) | **Blocking** — required before Concept → Spec |
| Recovery procedure for Create Loan + Disbursement Core Banking failure | Operations / Risk | Open Question 6 |
| HWM `CreateLoanDisbCash` hard block implementation in Underwriting Workflow execution step engine | Underwriting Workflow capability | Must be confirmed as implemented |

---

## NFRs Escalated to Capability

- Idempotency hard block on `CreateLoanDisbCash` → [CAPABILITY.md NFRs](../CAPABILITY.md)
- Skip guard on `CreateFacilityCash` → [CAPABILITY.md NFRs](../CAPABILITY.md)
- MongoDB audit payload storage → [CAPABILITY.md NFRs](../CAPABILITY.md)
- Atomicity of HWM + state transition → [CAPABILITY.md NFRs](../CAPABILITY.md)
