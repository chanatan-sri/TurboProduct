# Feature: Post-Disbursement QA (Matcha Callback)

**Parent Capability**: [Cash Disbursement](../CAPABILITY.md)
**Parent Product**: [Onigiri — Loan Origination System](../../../PRODUCT.md)
**Engineering Owner**: TBD
**Status**: Concept

---

## User Story

> As the **system**, I want to submit a document verification task to Matcha after cash is disbursed and route the application to `Funded`, `Draft`, or `pending_approval` based on Matcha's callback, so that document compliance is verified post-disbursement without blocking fund release.

---

## Job-to-be-Done

Cash is irreversible — funds have already left when this state is entered. Document verification here is a **post-disbursement compliance check**, not a disbursal gate. Matcha verifies the document package after disbursement and its outcome drives whether the loan is formally marked active (`Funded`), requires document correction (`Draft`), or needs re-assessment (`pending_approval`).

This is structurally the same Matcha integration as `pending_document_checking` in the non-cash path, but the target states differ because disbursement has already occurred.

---

## State Covered

| State | Role |
|-------|------|
| `QA_cash` | System submits document verification task to Matcha on entry; waits for callback; routes based on Matcha outcome |

---

## Acceptance Criteria

### Task Submission (on entry to `QA_cash`)

- [ ] On entry to `QA_cash`, Onigiri submits a document verification task to Matcha via `POST /task` with the application ID, document references, and cash disbursement context.
- [ ] HWM is updated to `QA_cash` on entry before the Matcha task is submitted.
- [ ] The Matcha task submission request and response are stored in MongoDB before the state machine waits.
- [ ] If the Matcha task submission fails (5xx), the application remains in `QA_cash`. Alert fires. Manual retry or investigation required.

### Callback Routing (on Matcha callback)

- [ ] On Matcha callback with outcome `APPROVED`: application transitions to `Funded` (terminal). Callback payload stored in MongoDB. HWM updated.
- [ ] On Matcha callback with outcome `RETURNED`: application transitions to `Draft`. The Matcha return reason (document deficiency details) is stored on the application record and surfaced to the loan officer. Callback payload stored in MongoDB.
- [ ] On Matcha callback with outcome `REFERRED`: application transitions to `pending_approval` for human re-assessment. Callback payload stored in MongoDB.
- [ ] All transitions are **atomic**: state transition and MongoDB payload write complete before 200 OK is returned to Matcha.
- [ ] Callbacks are **idempotent**: if a duplicate callback is received after the state has already advanced, it is a no-op. No error raised.
- [ ] A callback received when the application is no longer in `QA_cash` (e.g., already `Funded`) is silently ignored.

### Post-Disbursement Constraint

- [ ] **The loan in Core Banking remains active regardless of Matcha outcome.** `RETURNED` and `REFERRED` outcomes do not cancel or recall the disbursed cash. They trigger remediation on the Onigiri application record only.
- [ ] All financial fields and disbursement channel remain read-only (HWM locks `ALL_FINANCIAL` and `DISBURSEMENT_CHANNEL`).

---

## Edge Cases & Error States

| Scenario | Expected Behavior |
|----------|-------------------|
| Matcha returns `RETURNED` → application goes to Draft → documents re-uploaded → resubmitted | Application re-enters `QA_cash`. New Matcha task submitted. Previous task payload retained in MongoDB. |
| Matcha returns `REFERRED` → application goes to `pending_approval` → approver re-approves | Application re-enters cash path from `Cash?` decision. `QA_cash` will be reached again post-disbursement. |
| Matcha task submission fails (5xx) | Application stays in `QA_cash`. Alert fires. No auto-retry. Manual investigation. |
| Matcha callback timeout — no callback within SLA | Alert fires. SLA and escalation path TBD (Open Question — see CAPABILITY.md). |
| Duplicate Matcha callback received | Idempotency check: if application already left `QA_cash`, silently ignore. |
| Matcha returns unknown outcome code | Treat as `REFERRED` (fail safe to human review). Alert engineering. |

---

## Out of Scope

- Pre-disbursement document verification (`QA_noncash` → `pending_document_checking`) — owned by [Disbursement Orchestration](../../disbursement-orchestration/CAPABILITY.md)
- Matcha internal verification logic, checklist, or scoring — owned by Matcha product
- Physical cash recall or Core Banking disbursement reversal — outside Onigiri scope

---

## Comparison: Cash vs. Non-Cash Matcha Integration

| Aspect | Non-Cash (`pending_document_checking`) | Cash (`QA_cash`) |
|--------|----------------------------------------|------------------|
| Timing | Pre-disbursement | Post-disbursement |
| `APPROVED` → | `waiting_for_confirmation` or `waiting_create_facility` | `Funded` (terminal) |
| `RETURNED` → | `Draft` | `Draft` |
| `REFERRED` → | `pending_approval` | `pending_approval` |
| Fund impact of failure | Blocks disbursement | No impact — funds already released |

---

## Dependencies

| Dependency | Type | Status |
|------------|------|--------|
| Matcha `POST /task` API contract (cash context payload) | External (Matcha product) | Open — cash-specific task payload to be confirmed |
| Matcha callback payload schema (APPROVED / RETURNED / REFERRED) | External (Matcha product) | Existing pattern — confirm cash path uses same schema |
| `QA_cash` dwell SLA and escalation path | NFR (Operations) | Open |

---

## NFRs Escalated to Capability

- Idempotent callback handling → [CAPABILITY.md NFRs](../CAPABILITY.md)
- MongoDB payload storage for all Matcha interactions → [CAPABILITY.md NFRs](../CAPABILITY.md)
- QA dwell monitoring and alerting → [CAPABILITY.md NFRs](../CAPABILITY.md)
