# FEATURE F18: SAP Connector — Cancel JV Transaction

**Feature ID**: F18
**POB Ticket**: POB-2389
**Type**: New Feature
**Priority**: TBC
**Status**: Spec
**Parent Capability**: [SAP Connector](../CAPABILITY.md)
**Engineering Owner**: [TBC]
**Last Updated**: 2026-03-04

---

## User Story

As a system support user,
I want to cancel a JV transaction batch that has failed to post to SAP,
So that the underlying accounting transactions are released back to `Ungroup` status and can be re-grouped into a new, corrected JV batch.

---

## System Context

**As-Is:** No mechanism to cancel a JV batch once created. When a batch fails SAP posting, its transactions remain locked and cannot be re-processed.

**To-Be:** Authorized user can cancel a JV batch with Posting Status = `FAILED`. Voucher Status changes from `CONFIRMED` to `CANCELLED`. Linked transactions released back to `Ungroup`. `FAILED` Posting Status preserved as historical record.

**Components Impacted:** `sap_jv_transaction`, `sap_transaction`, `master_jv_transaction_voucher_status`, `master_jv_transaction_posting_status`, `master_group_lifecycle_status`

---

## Status Reference

### JV Transaction — Voucher Status

| Status | Definition | Triggered By |
|--------|-----------|-------------|
| `CONFIRMED` | Batch created and valid. Linked transactions locked. | System creates new JV batch. |
| `CANCELLED` | Batch cancelled. Linked transactions released for re-grouping. | Authorized user cancels a `FAILED` batch. |

### JV Transaction — Posting Status

| Status | Definition | Triggered By |
|--------|-----------|-------------|
| `PENDING` | Batch created, awaiting SAP upload. | System creates new JV batch. |
| `SUCCESS` | SAP confirmed posting. Terminal state. | SAP result received — success. |
| `FAILED` | SAP rejected. User intervention required. | SAP result received — failure. |

### Combined Workflow

```
Batch created      → Voucher: CONFIRMED  | Posting: PENDING
SAP upload success → Voucher: CONFIRMED  | Posting: SUCCESS (terminal — locked)
SAP upload failed  → Voucher: CONFIRMED  | Posting: FAILED
User cancels       → Voucher: CANCELLED  | Posting: FAILED (preserved as history)
                     Linked transactions → Grouping Status: Ungroup (released)
```

---

## Acceptance Criteria

**Scenario 1: Cancel a failed JV batch — releases transactions**

Given a JV batch with Voucher Status = `CONFIRMED` and Posting Status = `FAILED`,
When an authorized user triggers the cancel action,
Then: Voucher Status → `CANCELLED`, Posting Status remains `FAILED`, all linked transactions' Grouping Lifecycle Status → `Ungroup`.

**Scenario 2: Cancel rejected when Posting Status is not FAILED**

Given a JV batch with Posting Status = `PENDING` or `SUCCESS`,
When an authorized user attempts to cancel,
Then the system rejects with a validation error — cancel is only permitted when Posting Status = `FAILED`.

**Scenario 3: Released transactions available for re-grouping**

Given a JV batch that has been cancelled and its transactions reverted to `Ungroup`,
When the next JV grouping run executes,
Then those transactions are eligible to be included in a new JV batch.

---

## Edge Cases & Out-of-Scope

| Scenario | Behaviour |
|----------|-----------|
| Cancel a `SUCCESS` JV batch | Rejected — cannot cancel after successful SAP posting |
| Cancel a `PENDING` JV batch | Rejected — batch must have a FAILED result before cancellation |
| Re-cancelling an already `CANCELLED` batch | Reject — no action needed |
| FAILED Posting Status after cancellation | Preserved as historical record — not overwritten |

---

## Dependencies

| Dependency | Type |
|-----------|------|
| F16 (SAP Result Upload) — must set Posting Status to `FAILED` before cancel is allowed | Blocking |
| F14 (Outbound) — linked `sap_transaction` records are released back to `Ungroup` | Related |
| F15 / F17 (JV Batching) — released transactions re-enter the batch cycle | Downstream |
| F19 (Cancel Transaction) — alternative for cancelling individual transactions before batching | Related |

> **Note (Release 1):** Delivered as a backdoor API. Self-service cancel flow via AMS planned for a future release.
