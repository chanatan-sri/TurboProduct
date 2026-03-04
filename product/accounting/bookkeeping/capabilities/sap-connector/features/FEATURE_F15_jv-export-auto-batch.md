# FEATURE F15: SAP Connector — Journal Voucher Export (Auto Batch)

**Feature ID**: F15
**POB Ticket**: POB-2202
**Type**: New Feature
**Priority**: P2
**Status**: Spec
**Parent Capability**: [SAP Connector](../CAPABILITY.md)
**Engineering Owner**: [TBC]
**Last Updated**: 2026-03-04

---

## User Story

As the SAP posting process,
I want the Bookkeeping system to automatically consolidate unposted accounting transactions into JV files at T+1,
So that SAP FI receives a structured, validated JV upload file each business day without manual intervention.

---

## System Context

**As-Is:** JV files for SAP are created manually and are not generated from a structured accounting transaction store.

**To-Be:** A nightly batch job consolidates all `sap_transaction` records with Grouping Status `Ungroup` and active status `true` into JV Transactions. A JV Upload file (max 900 group documents per file) is generated and placed on S3.

**Components Impacted:** `sap_jv_transaction`, `sap_jv_transaction_detail`, `sap_config_jv_transaction`, `sap_config_jv_transaction_file`, `sap_file`, `sap_transaction`

---

## File Specification

**File Naming:** `NTBxxx_yyyymmdd` (e.g., `NTB001_20250814`)
**S3 Path:** `daily-report/Accounting/JVupload/Bookkeeping/auto/yyyy/mm/NTBxxx_yyyymmdd`
**Max Group Documents per File:** 900 — overflow creates a new file with running number restarting from 1
**Rows per group:** 2 (one DR line, one CR line) — must net to zero

### JV File Columns (43 total — key columns)

| # | Column | Required | Notes |
|---|--------|----------|-------|
| 1 | Grp Doc | Yes | Sequential per file; resets to 1 in new file if > 900 |
| 2 | Company Code | Yes | 4 digits: NTB=`1000`, NTBPL=`1010`, NTBI=`1020`, NTBX=`1030` |
| 4 | Posting Date | Yes | Format `ddmmyyyy` |
| 5 | Document Date | Yes | Format `ddmmyyyy` |
| 6 | Doc Type | Yes | `SA` (HQ) or `PT` (branch) |
| 7 | Currency | Yes | Always `THB` |
| 9 | Reference | Yes | Accounting Event Code (e.g., `BOPE001`) |
| 10 | Doc Header Text | Yes | `YYYYMMDDEventCode_000000NNN` — primary key for SAP result matching |
| 14 | Posting Key | Yes | Customer: DR=01, CR=11 / Vendor: DR=21, CR=31 / GL: DR=40, CR=50 |
| 15 | Account | Yes | GL account number (or Customer/Vendor number if sub-ledger) |
| 17 | Amount | Yes | Positive for DR keys (40,01,21); Negative for CR keys (50,31,11). All lines in group must sum to zero. |
| 23 | Cost Center | No | From DR/CR cost center on the accounting transaction |
| 24 | Profit Center | No | From DR/CR profit center on the accounting transaction |

### Grouping Attributes

Transactions are consolidated into one group when ALL match: Company Code, Posting Date, Document Date, Doc Type, Currency, Reference (Event Code), Business Place, Posting Key, Account, Baseline Date, Profit Center, Text.

### Data Flow Example

```
Source transactions (Event BPAY001, Company 1000):
  Case 6: DR 1110000000 +100,000 | CR 9111000021 -100,000
  Case 7: DR 1110000000  +55,000 | CR 9111000021  -55,000
  Case 8: DR 1110000000 +350,530 | CR 9111000021 -350,530

JV File output (Group Doc 4 — 2 rows, nets to zero):
  Row 1: PK 40  Account 1110000000  Amount +505,530.00
  Row 2: PK 50  Account 9111000021  Amount -505,530.00
```

---

## Acceptance Criteria

**Scenario 1: Batch groups eligible transactions and assigns reference numbers**

Given transactions with Grouping Status `Ungroup`, active status `true`, within target date range,
When the nightly batch runs,
Then the system groups by defined attributes, creates `sap_jv_transaction` records with Doc Header Text `YYYYMMDDEventCode_000000NNN`, and updates each transaction's Grouping Status to `Grouped`.

**Scenario 2: JV file generated and placed on S3**

Given JV transactions produced by the batch,
When file generation runs,
Then the system produces a 43-column file (max 900 group docs), named `NTBxxx_yyyymmdd`, stored at the auto batch S3 path.

**Scenario 3: File splits correctly when group count exceeds 900**

Given a batch producing 1,050 group documents,
When the file is generated,
Then two files are created: first with groups 1–900, second with groups 1–150 (running number restarts), each with distinct file names on S3.

**Scenario 4: All amounts within each group net to zero**

Given any generated JV file,
When file content is validated before S3 upload,
Then sum of all amounts within each group equals zero; any unbalanced group is flagged and excluded.

**Scenario 5: Already-grouped transactions are excluded**

Given a mix of `Grouped` and `Ungroup` transactions,
When the batch runs,
Then only `Ungroup` transactions are included.

---

## Open Items

| # | Question | Status |
|---|----------|--------|
| 1 | What is the batch schedule? Nightly cron at what time? | Open |
| 2 | Is the 900 doc group limit a SAP FI constraint or a Bookkeeping rule? | Open |
| 3 | What happens if balance validation fails for a group — excluded or batch fails? | Open |

---

## Dependencies

| Dependency | Type |
|-----------|------|
| F14 (Outbound) — `sap_transaction` records must exist with `Ungroup` status | Blocking |
| F16 (SAP Result) — updates statuses after file is uploaded to SAP | Downstream |
| F17 (Manual JV) — shares same grouping logic but triggered manually | Related |
