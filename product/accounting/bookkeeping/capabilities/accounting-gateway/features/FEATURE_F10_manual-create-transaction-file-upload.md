# FEATURE F10: Accounting Gateway — Manual Create Transaction by File Upload (AMS)

**Feature ID**: F10
**POB Ticket**: POB-2206
**Type**: Enhancement
**Priority**: P1
**Status**: Spec
**Parent Capability**: [Accounting Gateway](../CAPABILITY.md)
**Engineering Owner**: [TBC]
**Last Updated**: 2026-03-04

---

## User Story

As an accounting user (Preparer) on AMS,
I want to upload a structured file to create multiple manual accounting transactions at once,
So that I can record bulk journal entries efficiently without being bound by standard accounting event configuration.

## Job-to-be-done

Accounting users need to make manual adjustments and corrections that don't follow standard event templates. They supply GL accounts directly and need the system to validate them — giving them full control while enforcing COA rules.

**Key distinction from F8:** The user supplies DR and CR GL account numbers directly. The system does NOT perform event config lookup for GL resolution — it only uses the GL master to determine center type (profit/cost) and sub-ledger type.

---

## System Context

**As-Is:** Manual accounting adjustments are entered individually or managed outside the system.

**To-Be:** AMS provides a file upload screen where a Preparer submits a CSV file. System validates format, then validates each row against master data. Valid rows recorded to Accounting Book. Summary notification sent to Google Chat.

**Components Impacted:** `journal_header`, `journal_line`, `journal_line_attributes`, `journal_header_references`, `master_general_ledger`, `master_sub_ledger_type`, `master_organizational_unit`, AMS frontend

---

## File Format Specification

**Entry point:** AMS file upload screen (Preparer role)
**Columns:** 17

| # | Column Name | Required | Validation |
|---|------------|----------|-----------|
| 1 | Event code | Required | Must match an active Accounting Event |
| 2 | Posting date | Required | Format `YYYY-MM-DD` |
| 3 | Doc date | Required | Format `YYYY-MM-DD` |
| 4 | Dr Account Number | Required | Must match an active GL account in COA |
| 5 | Cr Account Number | Required | Must match an active GL account in COA |
| 6 | Dr. Profit center / cost center | Optional | If provided, must exist in COA. Type (profit/cost) determined by DR GL master config. |
| 7 | Cr. Profit center / cost center | Optional | If provided, must exist in COA. Type determined by CR GL master config. |
| 8 | Dr. Sub ledger | Optional | If provided, must be valid for DR GL. Ignored if DR GL is "No Sub Ledger". |
| 9 | Cr. Sub ledger | Optional | If provided, must be valid for CR GL. Ignored if CR GL is "No Sub Ledger". |
| 10 | Amount | Required | Decimal > 0, no comma |
| 11 | Reference1 | Required | Not null |
| 12–16 | Reference2–6 | Optional | — |
| 17 | Remark | Optional | Free-text note |

**File Example:**
```
Event code,posting date,Doc date,Dr Account Number,Cr Account Number,Dr. Profit center / cost center,Cr. Profit center / cost center,Dr. Sub ledger,Cr. Sub ledger,Amount,Reference1,Reference2,Reference3,Reference4,Reference5,Reference6,Remark
BPTC001,2026-01-27,2026-01-27,1110100000,1110110000,1005000100,1005012300,,150,NBI001,,,,,,,Manual entry
BPTC006,2026-01-27,2026-01-27,5211313000,1110100000,1005012300,1005012300,,200,NBI001,100,,,,,,Manual entry
```

---

## Processing Logic

**Stage 1 — File Format Validation (all-or-nothing):**
- All 17 column headers present
- Required fields not null
- Date format `YYYY-MM-DD`
- Amount is numeric and > 0
- If ANY row fails → entire file rejected, notification sent

**Stage 2 — Row-Level Master Data Validation (valid rows continue):**
Each row validated independently in order:
1. Event code exists and is active
2. DR Account Number exists in COA (active GL)
3. CR Account Number exists in COA (active GL)
4. If DR center provided → must exist in COA; GL master determines profit/cost type
5. If CR center provided → must exist in COA; GL master determines profit/cost type
6. If DR sub-ledger provided → must be valid for DR GL; silently ignored if DR GL = "No Sub Ledger"
7. If CR sub-ledger provided → must be valid for CR GL; silently ignored if CR GL = "No Sub Ledger"
8. Amount > 0
9. Date format valid

All errors per row are collected and concatenated.

### Validation Error Messages (Stage 2)

| Condition | Error Message |
|-----------|--------------|
| Required field null | `กรุณาระบุ '[Field]'` |
| Field format incorrect | `Format '[Field]' ไม่ถูกต้อง` |
| Event code not found | `ไม่พบ Event Code บนระบบ` |
| DR Account not found | `ไม่พบ Dr. Account บนระบบ` |
| CR Account not found | `ไม่พบ Cr. Account บนระบบ` |
| DR center not found | `ไม่พบ Dr. Profit center / cost center บนระบบ` |
| CR center not found | `ไม่พบ Cr. Profit center / cost center บนระบบ` |
| DR sub-ledger not found | `ไม่พบ Dr. Sub ledger บนระบบ` |
| CR sub-ledger not found | `ไม่พบ Cr. Sub ledger บนระบบ` |
| Amount negative/zero | `amount มีค่าติดลบ` |
| Amount not numeric | `Format amount ไม่ถูกต้อง` |
| Date format invalid | `Format 'posting date' ไม่ถูกต้อง` |

### Notification Format (Google Chat)

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

Given a file passing all Stage 1 checks and all rows reference valid master data,
When a Preparer uploads via AMS,
Then the system creates a journal record for each row, records uploader as `created_by`, and sends Google Chat notification.

**Scenario 2: File fails Stage 1 — entire file rejected**

Given a file with an incorrect date format (e.g., `14/02/2025`) or missing column,
When Stage 1 validation runs,
Then no transactions are created, entire file rejected, rejection notification sent.

**Scenario 3: File passes Stage 1 but rows fail Stage 2**

Given a file passing Stage 1, but certain rows reference non-existent event codes or GL accounts,
When processed row by row in Stage 2,
Then valid rows are recorded, invalid rows logged with specific errors, processing continues for all rows.

**Scenario 4: Multiple validation errors on a single row — all reported**

Given a row where event code doesn't exist AND DR center doesn't exist,
When Stage 2 validates that row,
Then both errors reported together: `ไม่พบ Event Code บนระบบ, ไม่พบ Dr. Profit center / cost center บนระบบ`.

**Scenario 5: Amount is zero or negative**

Given a row with Amount ≤ 0,
When validated in Stage 2,
Then row rejected with `amount มีค่าติดลบ`, remaining rows continue.

**Scenario 6: Sub ledger provided for a "No Sub Ledger" GL**

Given a row where DR GL is "No Sub Ledger" and a DR Sub Ledger value is provided,
When the row is processed,
Then the sub ledger value is silently ignored and the transaction is created without a sub ledger attribute on the DR side.

---

## Dependencies

| Dependency | Type |
|-----------|------|
| F2 (GL master) — validates DR/CR account numbers | Blocking |
| F4/F6 (Centers) — validates center codes | Blocking |
| F3 (Sub-ledger type) — determines sub-ledger validation | Blocking |
| F12 (Journal Record) — stores valid transactions | Downstream |
| F14 (SAP Outbound) — creates `sap_transaction` after save | Downstream |
