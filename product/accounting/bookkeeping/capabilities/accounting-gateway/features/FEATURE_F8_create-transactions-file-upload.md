# FEATURE F8: Accounting Gateway — Create Transactions by File Upload (Upstream)

**Feature ID**: F8
**POB Ticket**: POB-2198
**Type**: New Feature
**Priority**: P1
**Status**: Spec
**Parent Capability**: [Accounting Gateway](../CAPABILITY.md)
**Engineering Owner**: [TBC]
**Last Updated**: 2026-03-04

---

## User Story

As an upstream business system (e.g., Cash Reconciliation, automated EOD process),
I want to submit a batch of accounting transactions via a structured file upload,
So that multiple journal entries can be created in a single operation with the GL mapping resolved automatically from the accounting event configuration.

## Job-to-be-done

End-of-day batch processes generate many transactions at once. Upstream systems should not need to know GL account details — Bookkeeping owns the mapping logic. File upload enables bulk submission without real-time API call overhead.

**Key distinction from F10:** F8 does NOT require the submitter to specify GL accounts, centers, or sub-ledgers. These are derived entirely from event config. F10 (manual create) requires the submitter to specify them explicitly.

---

## System Context

**As-Is:** Bulk accounting data handled manually or through ad hoc imports with no standardized format or automated validation.

**To-Be:** System accepts a structured file, validates format first, then processes each row by resolving full double-entry mapping from event config. Valid rows are recorded to the Accounting Book. Summary notification sent on completion.

**Components Impacted:** `journal_header`, `journal_line`, `journal_line_attributes`, `journal_header_references`, `master_accounting_event`, `accounting_config_event`, `log_request`, `log_response`

---

## File Format Specification

**Entry point:** S3 file upload from upstream system
**Columns:** 11

| # | Column Name | Required | Validation |
|---|------------|----------|-----------|
| 1 | Event code | Required | Must match an active Accounting Event |
| 2 | Posting date | Required | Format `YYYY-MM-DD`, not null |
| 3 | Doc date | Required | Format `YYYY-MM-DD`, not null |
| 4 | Amount | Required | Decimal > 0, no comma separator |
| 5 | Reference1 | Required | Not null |
| 6 | Reference2 | Optional | — |
| 7 | Reference3 | Optional | — |
| 8 | Reference4 | Optional | — |
| 9 | Reference5 | Optional | — |
| 10 | Reference6 | Optional | — |
| 11 | Note | Optional | Free-text remark |

**File Example:**
```
Event code,posting date,Doc date,Amount,Reference1,Reference2,Reference3,Reference4,Reference5,Reference6,Note
BPTC001,2025-01-01,2025-01-01,100,NBI001,,,,,, test1
BPTC002,2025-01-01,2025-01-01,100,NBI001,,,,,, test2
```

---

## Processing Logic

**Stage 1 — File Format Validation (all-or-nothing):**
- All column names present and correctly named
- Required fields not null
- Date fields conform to `YYYY-MM-DD`
- Amount is a valid positive number
- If ANY row fails → entire file rejected, notification sent, no transactions created

**Stage 2 — Row-Level Validation (valid rows continue):**
- Event code exists and is active
- GL resolution from event config (DR/CR accounts, center, sub-ledger)
- Failed rows logged with error; valid rows continue processing

### Validation Error Messages (Stage 2)

| Condition | Error Message |
|-----------|--------------|
| Event code not found | `ไม่พบ Event Code บนระบบ` |
| Required field null | `กรุณาระบุ '[Field]'` |
| Field format incorrect | `Format '[Field]' ไม่ถูกต้อง` |
| Amount negative/zero | `amount มีค่าติดลบ` |
| Date format invalid | `Format 'posting date' ไม่ถูกต้อง` |

Multiple errors per row are concatenated.

### Notification Format

```
Request File name: [filename]
Total request: [N] รายการ
Success request: [N] รายการ
Fail request: [N] รายการ
File Result: [S3 result link]
```

---

## Acceptance Criteria

**Scenario 1: Valid file — all rows processed**

Given a file passing all Stage 1 checks and all rows reference valid active event codes,
When uploaded to the gateway,
Then the system creates a journal record for each row with GL mapping resolved from event config, and sends a summary notification.

**Scenario 2: File fails Stage 1 — entire file rejected**

Given a file where at least one row has a missing required column or incorrectly formatted date,
When Stage 1 validation runs,
Then no transactions are created, the entire file is rejected, and a rejection notification is sent.

**Scenario 3: File passes Stage 1 but individual rows fail Stage 2**

Given a file passing Stage 1, but certain rows reference a non-existent event code,
When processed row by row in Stage 2,
Then valid rows are recorded, invalid rows are logged with specific errors, and processing continues for all remaining rows.

**Scenario 4: Amount is zero or negative**

Given a row with Amount ≤ 0 that passes Stage 1,
When validated in Stage 2,
Then that row is rejected with `amount มีค่าติดลบ` and processing continues.

**Scenario 5: Processing summary sent on completion**

Given any file that completes processing,
When Stage 2 finishes,
Then the system sends a notification with file name, total rows, success count, fail count, and S3 result link.

---

## Edge Cases & Out-of-Scope

| Scenario | Behaviour |
|----------|-----------|
| GL accounts in file | Not expected — GL is always resolved from event config in this path |
| Data migration use case | Supported — historical data can be uploaded in this format |
| Re-upload of same file | Idempotency behaviour TBC |

---

## Dependencies

| Dependency | Type |
|-----------|------|
| F5 (Event Config) — GL resolution | Blocking |
| F4/F6 (Centers) — center lookup | Blocking |
| F12 (Journal Record) — stores valid transactions | Downstream |
| F14 (SAP Outbound) — creates `sap_transaction` after save | Downstream |
