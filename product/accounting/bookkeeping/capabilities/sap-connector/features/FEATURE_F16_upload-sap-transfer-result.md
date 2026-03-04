# FEATURE F16: SAP Integration — Upload SAP Transfer Result

**Feature ID**: F16
**POB Ticket**: POB-2624
**Type**: New Feature
**Priority**: P2
**Status**: Spec
**Parent Capability**: [SAP Connector](../CAPABILITY.md)
**Engineering Owner**: [TBC]
**Last Updated**: 2026-03-04

---

## User Story

As an accounting user on AMS,
I want to upload the SAP posting result file after JV submission,
So that the Bookkeeping system reflects the accurate Success/Fail status for each JV transaction group and the accounting team can act on failures promptly.

---

## System Context

**As-Is:** SAP posting results are tracked manually and do not feed back into the accounting system.

**To-Be:** AMS provides an upload screen for the SAP result file (CSV). System validates format, then processes each row — matching `Doc Header Text` to JV transaction groups, updating posting statuses, and sending a Google Chat summary.

**Components Impacted:** `sap_jv_transaction`, `sap_jv_transaction_detail`, `sap_file`, AMS frontend

---

## File Format Specification

**Format:** CSV | **Required Columns:** 5

| # | Column | Required | Validation |
|---|--------|----------|-----------|
| 1 | Row | Required | Positive integer, not null |
| 2 | Doc Header Text | Required | Format `YYYYMMDDEventCode_000000NNN`, not null, must exist in system |
| 3 | Status | Required | Exactly `Success` or `Fail` — no other values |
| 4 | Key | Required | `Debit` or `Credit` |
| 5 | Error Message | Optional | Free-text SAP error detail |

**File Example:**
```
Row,Doc Header Text,Status,Key,Error Message
1,20250828BOPE001_000000001,Fail,Debit,Invalid GL
2,20250828BOPE001_000000001,Fail,Credit,Invalid GL
3,20250828BOPE001_000000002,Success,Debit,
4,20250828BOPE001_000000002,Success,Credit,
```

**Status Resolution Rule:** A group is `Fail` if ANY row in that group is `Fail`. Group is `Success` only when ALL rows are `Success`. Each group must have both a Debit and Credit row — missing either side → `Key ไม่ครบ`.

---

## Processing Logic

**Stage 1 — File Format Validation (all-or-nothing):**
- All 5 column headers present
- Required fields not null
- Status = `Success` or `Fail` only
- Doc Header Text conforms to expected format
- If ANY row fails → entire file rejected, no statuses updated

**Stage 2 — Row-Level Data Validation (valid rows continue):**
- `Doc Header Text` must exist in the Bookkeeping system
- Status overwrites are supported (Success → Fail and Fail → Success both allowed)

### Validation Error Messages

| Condition | Error Message |
|-----------|--------------|
| Required field null | `กรุณาระบุ [Field]` |
| Doc Header Text not found | `ไม่พบ Doc Header Text บนระบบ` |
| Doc Header Text format invalid | `Format Doc Header Text ไม่ถูกต้อง` |
| Status not Success or Fail | `Format Status ไม่ถูกต้อง` |
| Group missing Debit or Credit row | `Key ไม่ครบ` |

### Notification Format (Google Chat)

```
Date: ddmmyyyy hh:mm:ss
Uploaded file name: [filename]
Message: รายการทั้งหมด [N] รายการ - สำเร็จ [N] รายการ, ไม่สำเร็จ [N] รายการ
Link: [S3 result link]
```

---

## Acceptance Criteria

**Scenario 1: Valid file updates all JV group statuses correctly**

Given a valid result file where all rows pass validation,
When uploaded via AMS,
Then the system updates each JV group to `Success` or `Fail` per the resolution rule, records error messages for `Fail` rows, and sends a Google Chat notification.

**Scenario 2: Any Fail row marks the entire group as Fail**

Given a JV group with both Debit and Credit rows carrying `Status = Fail`,
When the file is processed,
Then the group's posting status → `Fail` and error messages from both rows are recorded.

**Scenario 3: Status overwrite supported in both directions**

Given a JV group previously marked `Success`,
When a new result file marks the same group as `Fail`,
Then the system overwrites to `Fail`. The reverse (Fail → Success) is equally supported.

**Scenario 4: Doc Header Text not found**

Given a result file row whose `Doc Header Text` does not correspond to any JV group,
When Stage 2 processes that row,
Then the row is flagged with `ไม่พบ Doc Header Text บนระบบ` and remaining rows continue.

**Scenario 5: Invalid Status value**

Given a row where `Status = Pending`,
When validated,
Then row rejected with `Format Status ไม่ถูกต้อง`, remaining rows continue.

**Scenario 6: File format failure rejects entire file**

Given a file with missing required column headers,
When Stage 1 validation runs,
Then entire file rejected, no statuses updated, user receives `Column ไม่ครบถ้วน, กรุณาตรวจสอบ Format`.

**Scenario 7: Group missing Debit or Credit row**

Given a JV group with only a Debit row (no Credit row) in the result file,
When the file is processed,
Then the group is flagged with `Key ไม่ครบ` and its status is not updated.

---

## Dependencies

| Dependency | Type |
|-----------|------|
| F15 (JV Auto Batch) or F17 (Manual JV) — JV groups must exist | Blocking |
| F13 (Pivot View) — refresh triggered after result update | Downstream |
| F18 (Cancel JV) — user action required after `Fail` result | Related |
