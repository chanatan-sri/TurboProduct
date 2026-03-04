# FEATURE F14: SAP Connector — Accounting Transaction Outbound

**Feature ID**: F14
**POB Ticket**: POB-2191
**Type**: New Feature
**Priority**: P1
**Status**: Spec
**Parent Capability**: [SAP Connector](../CAPABILITY.md)
**Engineering Owner**: [TBC]
**Last Updated**: 2026-03-04

---

## User Story

As the SAP integration process,
I want every accounting transaction created in Bookkeeping to have a tracked outbound status,
So that the system knows which transactions are pending SAP posting and can group them efficiently into JV transactions.

## Job-to-be-done

The SAP Connector needs visibility into every transaction from the moment it is created. Without a `sap_transaction` record, the JV batching process (F15) cannot determine which transactions are eligible for SAP posting.

---

## System Context

**As-Is:** No formal tracking of which journal entries have been submitted to SAP or their posting outcome.

**To-Be:** When an accounting transaction is created on the Accounting Book, a corresponding `sap_transaction` record is created with initial grouping status `Ungroup` (or `Do Not Post` for adjustment/migration transactions). Status updated as the transaction progresses through JV grouping and SAP posting.

**Components Impacted:** `sap_transaction`, `sap_transaction_history`, `master_inherited_posting_status`, `master_group_lifecycle_status`, `journal_header`

---

## Status Model

| Status Field | Initial Value | Changes By |
|---|---|---|
| Grouping Lifecycle Status | `Ungroup` (normal) / `Do Not Post` (SAP Record flag = N) | F15/F17 (batch), F18 (cancel batch), F19 (cancel transaction) |
| Inherited Posting Status | `Ungroup` (normal) / `Do Not Post` (excluded) | F16 (SAP result upload) |

---

## Acceptance Criteria

**Scenario 1: Normal transaction creates outbound record with Ungroup status**

Given a standard accounting transaction created via API or file upload,
When the journal record is written to the Accounting Book,
Then a `sap_transaction` record is created with Grouping Lifecycle Status = `Ungroup` and Inherited Posting Status = `Ungroup`.

**Scenario 2: Adjustment/migration transaction creates outbound record with Do Not Post status**

Given an accounting transaction flagged as type `adjustment` or `migration` (SAP Record flag = N),
When the journal record is written,
Then the `sap_transaction` record is created with Grouping Lifecycle Status = `Do Not Post`.

**Scenario 3: Status history is maintained**

Given a `sap_transaction` record that changes status over time (e.g., Ungroup → Grouped → Posted),
When each status change occurs,
Then a new record is appended to `sap_transaction_history` capturing the previous state and timestamp.

---

## Dependencies

| Dependency | Type |
|-----------|------|
| F12 (Journal Record) — `sap_transaction` is created alongside the journal record | Blocking |
| F15 (JV Auto Batch) — reads `sap_transaction` records with `Ungroup` status | Downstream |
| F16 (SAP Result) — updates Inherited Posting Status | Downstream |
| F18 (Cancel JV) — releases transactions back to `Ungroup` | Downstream |
| F19 (Cancel Transaction) — sets status to `Do Not Post` | Downstream |
