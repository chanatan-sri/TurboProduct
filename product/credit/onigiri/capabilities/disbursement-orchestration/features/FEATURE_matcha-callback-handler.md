# Feature: Matcha Callback Handler (Approved Path)

**Capability**: Disbursement Orchestration — [CAPABILITY](../CAPABILITY.md)
**Product**: Onigiri
**Engineering Owner**: TBD
**Status**: Concept
**Last Updated**: 2026-03-04

---

## User Story

> *"As the Onigiri system, I want to receive a Matcha `approved` callback and transition the loan application from `pending_document_checking` to `waiting_fund_transfer`, so that the disbursement pipeline can proceed automatically once document verification is confirmed — without manual intervention."*

---

## Job-to-be-Done

The loan application is blocked, waiting for an external document verification decision. When Matcha's QA confirms the documents are approved, Onigiri must atomically advance to the next stage without human intervention and without risk of double-transition if the callback is delivered more than once (e.g., after a network timeout where Onigiri did process it successfully).

Additionally, if Matcha later issues a re-decision (due to a late car check result), the application in `waiting_fund_transfer` must be correctly routed: reversed to `returned_for_revision`, escalated back to `pending_approval`, or left unchanged — depending on the re-decision outcome.

---

## Acceptance Criteria

All criteria are pass/fail verifiable.

| # | Given | When | Then |
|---|-------|------|------|
| AC-1 | Application in `pending_document_checking`. `matcha_task_uuid` on record matches callback `taskUuid`. | POST `/api/credit-application/verification-callback` with `outcome=approved`, `isReDecision=false`. | Application transitions to `waiting_fund_transfer` in a single PG transaction. Callback payload stored in MongoDB. Response: 200 OK. |
| AC-2 | Application in `pending_document_checking`. Same `taskUuid` + completion event ID has already been processed (duplicate delivery). | Same callback arrives again. | No state change. Response: 200 OK. (Idempotent.) |
| AC-3 | Application in `waiting_fund_transfer`. | POST with `outcome=returned`, `isReDecision=true`. | Application transitions to `returned_for_revision`. Payload stored in MongoDB. Response: 200 OK. |
| AC-4 | Application in `waiting_fund_transfer`. | POST with `outcome=referred`, `isReDecision=true`. | Application transitions to `pending_approval`. CA work-entry published via Raijin (processId=3, CA role) within the same PG transaction. Payload stored in MongoDB. Response: 200 OK. |
| AC-5 | Application in `waiting_fund_transfer`. | POST with `outcome=approved`, `isReDecision=true`. | No state change (application is already in the correct post-approval state). Payload stored in MongoDB for audit. Response: 200 OK. |
| AC-6 | Application in any state other than `pending_document_checking` (for initial) or `waiting_fund_transfer` (for re-decision). | Any Matcha callback arrives. | No state change. Payload stored in MongoDB for audit. Response: 200 OK. |
| AC-7 | Callback `taskUuid` does not match `matcha_task_uuid` stored on the application. | Any Matcha callback arrives. | No state change. No payload stored. Response: 404. |
| AC-8 | Application in `pending_document_checking` with valid `taskUuid`. Onigiri's PG write fails mid-transition. | POST with `outcome=approved`, `isReDecision=false`. | State is not partially updated. Response: 5xx. (Allows Matcha's retry logic — 3× exponential backoff — to redeliver.) |

---

## Edge Cases and Error States

| Case | Behaviour |
|------|-----------|
| **Duplicate callback after successful processing** | Matcha retries because its HTTP client timed out, but Onigiri had already committed. Detected by idempotency key (taskUuid + completion event ID) in RDS. No-op, 200 OK. |
| **State mismatch on re-decision** | Application already transitioned to `returned_for_revision` before re-decision callback arrives (e.g., due to a supervisor recall). Callback arrives with `isReDecision=true` on application that is no longer in `waiting_fund_transfer`. Falls under AC-6: no-op, payload stored, 200 OK. |
| **`triggerReason=car_check_review`** | Valid path. A Haibara-triggered car check result caused Matcha's PENDING_REVIEW re-entry. The same re-decision routing table applies regardless of `triggerReason`. |
| **Callback for `returned` or `referred` on `pending_document_checking`** | This is out of scope for this feature. These outcomes from `pending_document_checking` are owned by the Underwriting Workflow capability. The callback router must not invoke this feature's handler for those outcome/state combinations. |
| **`isReDecision=true` on application in `pending_document_checking`** | Unexpected — a re-decision should only reach an application that has already advanced past initial verification. Treat as AC-6: no-op, payload stored, 200 OK. Flag in monitoring. |
| **Malformed payload (missing required fields)** | Return 400. Do not store payload. Do not change state. |

---

## Dependencies

| Dependency | Type | Notes |
|---|---|---|
| Underwriting Workflow — 4-Phase State Machine feature | Intra-product | `waiting_fund_transfer` must be a valid state in the state machine with valid inbound (from `pending_document_checking`) and outbound transitions. |
| Matcha — `POST /api/credit-application/verification-callback` contract | External (Matcha product) | Defined in `product/operations/matcha/ARCHITECTURE.md`. This feature is a consumer of that contract. |
| Raijin work-entry publisher | Cross-cutting infrastructure | Required for AC-4: publish CA work-entry (processId=3) when `referred` re-decision received. Same mechanism used by existing Underwriting Workflow transitions. |
| Shared Matcha callback router | Intra-product | The physical endpoint is shared with the Underwriting Workflow capability (which owns `returned` and `referred` from `pending_document_checking`). The router must dispatch by current application state + outcome to the correct capability handler. |

---

## Status

`Concept` — user story and acceptance criteria defined. Not yet ready for engineering estimation. Blocked on:
- Confirmation of the Raijin publisher mechanism (same as existing `referred` flow in Underwriting Workflow)
- Engineering owner assignment
