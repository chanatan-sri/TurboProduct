# FEATURE F19: Cancel Accounting Transaction

**Feature ID**: F19
**POB Ticket**: POB-2417
**Type**: New Feature
**Priority**: TBC
**Status**: Spec
**Parent Capability**: [Accounting Book](../CAPABILITY.md)
**Engineering Owner**: [TBC]
**Last Updated**: 2026-03-04

---

## User Story

As a system support user,
I want to cancel an individual accounting transaction that is erroneous,
So that it is permanently excluded from JV grouping and SAP posting without requiring cancellation of an entire JV batch.

## Job-to-be-done

When a single transaction is wrong, the user should not have to cancel an entire JV batch (F18) to fix it. This feature provides surgical cancellation of individual transactions before they are grouped.

---

## System Context

**As-Is:** No mechanism to mark individual transactions as invalid. An erroneous transaction can only be excluded by cancelling the entire JV batch.

**To-Be:** Authorized user can cancel an individual transaction. Sets Data Integrity Status to `Void` and Grouping Lifecycle Status to `Do Not Post`, permanently excluding it from JV grouping and SAP posting.

**Components Impacted:** `journal_header`, `sap_transaction`, `sap_transaction_history`, `master_group_lifecycle_status`, `master_inherited_posting_status`

---

## Status Reference

### Data Integrity Status (on `journal_header`)

| Status | Definition | Triggered By |
|--------|-----------|-------------|
| `Active` | Default. Transaction is valid and eligible for all processing. | New transaction created. |
| `Void` | Transaction flagged as erroneous. Permanently excluded. | Authorized user cancels the transaction. |

### Grouping Lifecycle Status (on `sap_transaction`)

| Status | Definition | Triggered By |
|--------|-----------|-------------|
| `Ungroup` | Available for JV batching. | New transaction created; or JV batch cancelled (F18). |
| `Grouped` | Locked in a JV batch. | System includes transaction in a JV batch. |
| `Do Not Post` | Permanently excluded from JV batching. | Transaction created as non-SAP type; or this cancel action. |

### Inherited Posting Status (on `sap_transaction`)

| Status | Definition |
|--------|-----------|
| `Ungroup` | Not yet batched. |
| `Submitted` | Parent JV batch sent to SAP, awaiting response. |
| `Posted` | Parent JV batch successfully posted. |
| `Failed` | Parent JV batch failed. |
| `Do Not Post` | Excluded from SAP permanently — set by this cancel action. |

---

## Acceptance Criteria

**Scenario 1: Cancel an ungrouped accounting transaction**

Given an accounting transaction with Data Integrity Status = `Active` and Grouping Lifecycle Status = `Ungroup`,
When an authorized user cancels the transaction,
Then: Data Integrity Status → `Void`, Grouping Lifecycle Status → `Do Not Post`, Inherited Posting Status → `Do Not Post`.

**Scenario 2: Cancelled transaction is excluded from JV grouping**

Given an accounting transaction with Grouping Lifecycle Status = `Do Not Post`,
When the next JV grouping run executes,
Then the transaction is not included in any new JV batch.

**Scenario 3: Cancel is rejected for a transaction already in a JV batch**

Given an accounting transaction with Grouping Lifecycle Status = `Grouped`,
When an authorized user attempts to cancel it,
Then the system rejects with a validation error — the parent JV batch must be cancelled first (F18) before the individual transaction can be cancelled.

**Scenario 4: Cancelled transaction reflected in pivot view as Do Not Post**

Given a cancelled transaction with Inherited Posting Status = `Do Not Post`,
When the pivot view is queried,
Then the transaction's status is reflected as `Do Not Post` and it is not in any pending SAP upload queue.

---

## Edge Cases & Out-of-Scope

| Scenario | Behaviour |
|----------|-----------|
| Cancel a `Do Not Post` transaction (already excluded) | Reject — no action needed |
| Cancel a `Posted` / `Success` transaction | Out of scope — cannot cancel after SAP posting; requires a reversing entry |
| Undo a cancellation | Not supported — cancellation is permanent |

---

## Dependencies

| Dependency | Type |
|-----------|------|
| F12 (Journal Record) — sets `is_active` flag | Related |
| F18 (Cancel JV) — must be done first if transaction is `Grouped` | Related |
| F13 (Pivot View) — refresh triggered after cancellation | Downstream |

> **Note (Release 1):** Delivered as a backdoor API. Self-service cancel via AMS planned for future release.
