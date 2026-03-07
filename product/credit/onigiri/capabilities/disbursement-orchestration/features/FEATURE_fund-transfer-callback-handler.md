# Feature: Fund Transfer Callback Handler

**Capability**: Disbursement Orchestration — [CAPABILITY](../CAPABILITY.md)
**Product**: Onigiri
**Engineering Owner**: TBD
**Status**: Concept
**Last Updated**: 2026-03-04

---

## User Story

> *"As a loan officer, I want to confirm loan payment to the customer so that the application transitions from `waiting_for_confirmation` to `waiting_fund_transfer` and the disbursement can proceed."*
>
> *"As the Onigiri system, I want to receive a Core Banking fund transfer success callback and transition the loan application from `waiting_fund_transfer` to `waiting_create_loan_operation`, so that the loan operation creation step can proceed after the funds have been confirmed transferred."*

---

## State Flow

```
waiting_for_confirmation
        │
        │  [Loan officer calls Confirm Loan Payment]
        ▼
waiting_fund_transfer
        │
        │  [Core Banking POSTs fund transfer success callback]
        ▼
waiting_create_loan_operation
```

---

## Job-to-be-Done

Before funds can be transferred, a loan officer must explicitly confirm the loan payment to the customer. This confirmation step is the gate that advances the application from `waiting_for_confirmation` to `waiting_fund_transfer`. Once in `waiting_fund_transfer`, Onigiri awaits an asynchronous callback from Core Banking. A successful callback signals money has moved and the loan operation record can now be created. The callback is the sole trigger for that transition — Onigiri must not advance the state without explicit external confirmation.

---

## Acceptance Criteria

All criteria are pass/fail verifiable.

### Confirm Loan Payment Function

| # | Given | When | Then |
|---|-------|------|------|
| AC-1 | Application in `waiting_for_confirmation`. | Loan officer calls `POST /applications/{id}/confirm-loan-payment` with valid authentication. | Application transitions to `waiting_fund_transfer` in a single PG transaction. Confirmation timestamp and officer ID recorded. Response: 200 OK. |
| AC-2 | Application in `waiting_for_confirmation`. | Same officer or a different officer calls `POST /applications/{id}/confirm-loan-payment` again (duplicate). | No state change. Response: 200 OK. (Idempotent.) |
| AC-3 | Application in any state other than `waiting_for_confirmation`. | Loan officer calls `POST /applications/{id}/confirm-loan-payment`. | No state change. Response: 409 Conflict. |
| AC-4 | Request is missing authentication or caller is not an authorised loan officer role. | `POST /applications/{id}/confirm-loan-payment` is called. | Request rejected. Response: 401/403. No state change. |
| AC-5 | Application ID does not exist. | `POST /applications/{id}/confirm-loan-payment` is called. | Response: 404. No state change. |

### Fund Transfer Callback

| # | Given | When | Then |
|---|-------|------|------|
| AC-6 | Application in `waiting_fund_transfer`. | Core Banking POSTs fund transfer callback with `result=success` and a valid `transferReferenceId`. | Application transitions to `waiting_create_loan_operation` in a single PG transaction. Callback payload stored in MongoDB. Response: 200 OK. |
| AC-7 | Application in `waiting_fund_transfer`. Same `transferReferenceId` has already been processed (duplicate delivery). | Same callback arrives again. | No state change. Response: 200 OK. (Idempotent, dedup by `transferReferenceId`.) |
| AC-8 | Application in `waiting_fund_transfer`. | Core Banking POSTs callback with `result=failure`. | No state transition. Failure payload stored in MongoDB. Operational alert raised (mechanism TBD). Response: 200 OK. |
| AC-9 | Application in any state other than `waiting_fund_transfer`. | Any Core Banking fund transfer callback arrives. | No state change. Payload stored in MongoDB for audit. Response: 200 OK. |
| AC-10 | Application in `waiting_fund_transfer`. Onigiri's PG write fails mid-transition. | Core Banking POSTs callback with `result=success`. | State is not partially updated. Response: 5xx. (Allows Core Banking retry.) |
| AC-11 | Callback payload references an application ID that does not exist in Onigiri. | Any Core Banking fund transfer callback arrives. | No state change. No payload stored. Response: 404. |
| AC-12 | Callback arrives without valid authentication credentials. | Any Core Banking fund transfer callback arrives. | Request rejected. Response: 401. No state change. No payload stored. |
| AC-13 | Callback payload is missing `transferReferenceId` or `result`. | Malformed callback arrives. | Request rejected. Response: 400. No state change. No payload stored. |

---

## Edge Cases and Error States

| Case | Behaviour |
|------|-----------|
| **Confirm called while application is not in `waiting_for_confirmation`** | Application may have already been confirmed (idempotent, 200 OK) or is in an incompatible state (409 Conflict). No state change in either case. |
| **Confirm called after application is rejected or cancelled** | Application is not in `waiting_for_confirmation`. Falls under AC-3: 409 Conflict. |
| **Duplicate success callback** | Core Banking retries after network timeout, but Onigiri had already committed. Detected by `transferReferenceId` idempotency key in RDS. No-op, 200 OK. |
| **Failure followed by success** | Core Banking sends `failure` (e.g., insufficient float), then `success` after internal retry. If application is still in `waiting_fund_transfer` when `success` arrives, AC-1 applies normally. The `failure` record in MongoDB is retained for audit. |
| **Callback received after Matcha re-decision moved the application** | A Matcha re-decision (`returned`) transitioned the application from `waiting_fund_transfer` to `returned_for_revision` before Core Banking sent its callback. Core Banking's callback now finds the application in `returned_for_revision`. Falls under AC-4: no-op, payload stored, 200 OK. |
| **Repeated failure callbacks** | Each `failure` callback is stored in MongoDB. Each triggers an operational alert. The alert system is responsible for deduplication to prevent alert fatigue (mechanism TBD). |
| **Authentication mechanism** | TBD — candidates are shared secret (HMAC-SHA256 header), mutual TLS, or IP allowlist. Must be resolved before this feature can advance from Concept → Spec. Until defined, AC-7 cannot be implemented. |

---

## Out of Scope

- **What triggers the fund transfer itself.** This feature covers only the callback receipt. Whether Onigiri initiates the fund transfer on entering `waiting_fund_transfer` (by calling Core Banking) is a separate feature not yet specified. See Capability Open Question #1.
- **Operator retry workflow for failed transfers.** This feature stores the failure and raises an alert. The operator's recovery action (manual retry, application state reversion, etc.) is out of scope for this spec.
- **What happens after `waiting_create_loan_operation`.** The next state transition is not yet defined. See Capability Open Question #2.

---

## Open Dependency: Core Banking Fund Transfer Callback API Contract

The Core Banking fund transfer callback API contract **does not yet exist**. This feature creates the requirement. The following must be defined jointly with the Core Banking team before this feature can advance from Concept → Spec:

- Endpoint path and HTTP method for Onigiri to receive the callback
- Request payload schema: minimum required fields including `transferReferenceId`, `applicationId` (or equivalent Onigiri reference), `result` (enum: `success` / `failure`), `failureReason` (optional)
- Authentication mechanism (see Edge Cases above)
- Retry policy: how many retries, what backoff, on what HTTP response codes
- SLA: maximum time between fund transfer completion and callback delivery

---

## Dependencies

| Dependency | Type | Notes |
|---|---|---|
| Underwriting Workflow — 4-Phase State Machine feature | Intra-product | `waiting_for_confirmation`, `waiting_fund_transfer`, and `waiting_create_loan_operation` must be valid states in the state machine. |
| Core Banking — fund transfer callback API contract | External (Core Banking product) | **Not yet defined.** This feature creates the requirement. See Open Dependency above. |
| Core Banking PRODUCT.md — SENDS section | Cross-product documentation | Core Banking's product boundary must declare this as an outbound integration. See `product/platform/core-banking/PRODUCT.md`. |

---

## Status

`Concept` — user story and acceptance criteria defined. **Blocked on:**
- Core Banking fund transfer callback API contract (does not yet exist)
- Authentication mechanism selection
- Operational alert mechanism definition (for AC-3)
- Engineering owner assignment
