# Capability: Disbursement Orchestration

**Product**: Onigiri — [PRODUCT](../../PRODUCT.md)
**Portfolio**: Credit
**Product Owner**: TBD (Credit PO)
**Status**: 📝 Draft — @FEATURE decomposition in progress
**Last Updated**: 2026-03-04

---

## Business Function

Receive external system callbacks that confirm document verification approval and fund transfer success, and advance the loan application through the final pre-disbursement states — from post-document-verification through fund transfer confirmation to loan operation creation readiness.

## Why It Exists (First Principles)

- **Observable Progress**: Business stakeholders must distinguish "documents approved" from "funds transferred" from "loan operation created." These are three discrete business events, each representing a commitment from a different external system. Collapsing them into a single opaque state hides failure points and makes SLA measurement impossible.
- **Separation of Concerns**: Post-document-verification orchestration involves distinct external dependencies (Core Banking fund transfer) that are entirely separate from underwriting logic. Conflating this with the Underwriting Workflow capability would blur the capability boundary and misattribute ownership.
- **Auditability**: Each external callback represents a financial commitment. The transitions triggered by these callbacks must be atomic, idempotent, and independently auditable — properties that are easier to guarantee when the capability owns a narrow, well-defined state scope.

---

## Feature Inventory

| Feature | Status | Description |
|---------|--------|-------------|
| [Matcha Callback Handler (Approved Path)](features/FEATURE_matcha-callback-handler.md) | Concept | Receives Matcha `approved` outcome from `pending_document_checking`; transitions to `waiting_fund_transfer`. Also owns re-decision routing from `waiting_fund_transfer`. |
| [Fund Transfer Callback Handler](features/FEATURE_fund-transfer-callback-handler.md) | Concept | Receives Core Banking fund transfer result; transitions from `waiting_fund_transfer` to `waiting_create_loan_operation` on success. |

---

## Business Rules

### Decision Table 1: Matcha Initial Callback Routing

Applies when: application is in state `pending_document_checking`.

| Current State | `outcome` | `isReDecision` | Action |
|---|---|---|---|
| `pending_document_checking` | `approved` | `false` | Transition → `waiting_fund_transfer`. Store payload in MongoDB. |
| `pending_document_checking` | `returned` | `false` | **OUT OF SCOPE** — owned by Underwriting Workflow capability. |
| `pending_document_checking` | `referred` | `false` | **OUT OF SCOPE** — owned by Underwriting Workflow capability. |
| Any state ≠ `pending_document_checking` | any | `false` | No-op. Store payload in MongoDB for audit. Return 200 OK. |

---

### Decision Table 2: Matcha Re-Decision Routing

Applies when: application is in state `waiting_fund_transfer` and `isReDecision=true` (Matcha PENDING_REVIEW re-entry after car check late arrival).

| Current State | `outcome` | `isReDecision` | Action |
|---|---|---|---|
| `waiting_fund_transfer` | `approved` | `true` | No-op (application is already in the correct post-approval state). Store payload for audit. Return 200 OK. |
| `waiting_fund_transfer` | `returned` | `true` | Transition → `returned_for_revision`. Store payload in MongoDB. Return 200 OK. |
| `waiting_fund_transfer` | `referred` | `true` | Transition → `pending_approval`. Publish CA work-entry via Raijin (processId=3, CA role). Store payload in MongoDB. Return 200 OK. |
| Any state ≠ `waiting_fund_transfer` | any | `true` | No-op. Store payload in MongoDB for audit. Return 200 OK. |

> **Note:** `triggerReason` (`initial_submit` / `car_check_review`) does not affect routing logic. The same table applies regardless of trigger reason.

---

### Decision Table 3: Core Banking Fund Transfer Callback Routing

Applies when: Core Banking POSTs a fund transfer result callback to Onigiri.

| Current State | CB `result` | Action |
|---|---|---|
| `waiting_fund_transfer` | `success` | Transition → `waiting_create_loan_operation`. Store payload in MongoDB. Return 200 OK. |
| `waiting_fund_transfer` | `failure` | No state transition. Store failure payload in MongoDB. Raise operational alert (mechanism TBD). Return 200 OK. |
| Any state ≠ `waiting_fund_transfer` | any | No-op. Store payload in MongoDB for audit. Return 200 OK. |

---

## State Flow Diagram

```mermaid
stateDiagram-v2
    direction LR

    state "Decision Phase (partial)" as dec {
        DocCheck: pending_document_checking
        WaitFund: waiting_fund_transfer
        WaitLoan: waiting_create_loan_operation
    }

    state "Terminal" as term {
        ReturnedForRevision: returned_for_revision
        PendingApproval: pending_approval
        NextTBD: [next state — TBD]
    }

    DocCheck --> WaitFund : Matcha approved\n(isReDecision=false)
    WaitFund --> WaitLoan : Core Banking\nfund transfer success
    WaitFund --> ReturnedForRevision : Matcha returned\n(isReDecision=true)
    WaitFund --> PendingApproval : Matcha referred\n(isReDecision=true)
    WaitLoan --> NextTBD
```

---

## NFRs

| NFR | Requirement |
|-----|-------------|
| Transition atomicity | Each callback-triggered state transition is written to RDS in a single PG transaction. Partial transitions must not be recorded. MongoDB payload write may be in a separate transaction but must complete before returning 200. |
| Idempotency | Duplicate callbacks (same Matcha `taskUuid` + completion event ID; same Core Banking transfer reference ID) must be detected and no-op'd. Returns 200 OK. Idempotency key stored in RDS alongside transition record. |
| Auditability | Every callback payload (including no-op and failure cases) must be stored in MongoDB. RDS transition log records: actor = system, trigger = callback type, external reference ID, timestamp. |
| Callback failure tolerance | Onigiri returns 5xx only for genuine internal processing failures. State mismatches, idempotent duplicates, and out-of-scope callbacks all return 200 OK to prevent external systems from entering retry loops on non-retriable conditions. |
| Failure state visibility | A Core Banking `failure` result received while in `waiting_fund_transfer` must produce an observable alert. The application must not silently remain in `waiting_fund_transfer` without operator visibility. Alert mechanism TBD (see Open Questions). |

---

## Open Questions

1. **What triggers the fund transfer?** The user story for `FEATURE_fund-transfer-callback-handler` covers receiving the callback. It is not yet specified whether Onigiri initiates the fund transfer call to Core Banking on entering `waiting_fund_transfer`, or whether an external actor (e.g., a branch officer action in Sensei) triggers it. This must be resolved before this capability can advance from Concept → Spec.

2. **What is the next state after `waiting_create_loan_operation`?** Based on the existing PRODUCT.md state machine, the downstream path likely involves a QA or Confirmation step before `funded`. The exact target state and the cash/non-cash path split (if it applies here) must be defined. This is the primary open question blocking full specification of this capability.

3. **Recovery path for Core Banking fund transfer failure.** If Core Banking returns `failure`, what is the operator recovery action? Can the transfer be retried without restarting the application? Is there a maximum retry count? Does the application revert to a prior state or stay in `waiting_fund_transfer`? The alerting mechanism is also undefined.

4. **Timeout on `waiting_fund_transfer`.** If Core Banking does not callback within N hours/days, should the application be auto-escalated or expired? If yes, the timeout value and escalation target must be defined (similar to Draft state auto-expiry).
