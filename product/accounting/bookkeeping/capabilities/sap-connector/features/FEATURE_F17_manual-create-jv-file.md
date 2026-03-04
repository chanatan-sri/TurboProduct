# FEATURE F17: SAP Connector — Manual Create JV File (AMS)

**Feature ID**: F17
**POB Ticket**: POB-2363
**Type**: New Feature
**Priority**: TBC
**Status**: Spec
**Parent Capability**: [SAP Connector](../CAPABILITY.md)
**Engineering Owner**: [TBC]
**Last Updated**: 2026-03-04

---

## User Story

As an accounting user (Preparer) on AMS,
I want to manually trigger JV transaction creation for a selected company, date range, and set of event codes,
So that I can generate a JV Upload file on demand without waiting for the nightly batch, for cases requiring immediate or ad hoc SAP posting.

---

## System Context

**As-Is:** JV file creation is only possible via the automated nightly batch job.

**To-Be:** AMS provides a "Create JV" screen. User configures company code, posting date range, and event codes. System consolidates eligible transactions, creates JV transactions, generates files (max 900 group docs/file), stores on S3 under the Manual JV folder, and sends Google Chat notification.

**Components Impacted:** `sap_jv_transaction`, `sap_config_jv_transaction`, `sap_file`, `sap_transaction`, AMS frontend

---

## AMS Screen & User Flow

| Step | Action |
|------|--------|
| 1 | Select **Company Code** (dropdown: `1000` NTB / `1010` NTBPL / `1020` NTBI / `1030` NTBX) |
| 2 | Select **Posting Date range** (date pickers: From / To) |
| 3 | Select **Event Codes** (multi-select list filtered by company) |
| 4 | Review **summary table** — eligible transactions grouped by Company Code, Event Code, Posting Date, Doc Date with summed amounts |
| 5 | Click **Confirm** — confirmation dialog appears before processing |

**Grouping logic:** Same as F15 auto batch (by Company Code, Event Code, Posting Date, Doc Date).

---

## File Naming Convention

```
cccc_yyyymmdd_hhmmss_manual
```

Multiple files from one request get sequential timestamps (1 second apart):
```
1000_20251024_100143_manual
1000_20251024_100144_manual
1000_20251024_100145_manual
```

**S3 Path (Manual):** Separate from auto batch path. Downloadable from AMS JV display screen.

---

## Notification Format (Google Chat)

**Success:**
```
Date: ddmmyyyy hh:mm:ss
File name:
1000_20251024_100143_manual.txt
Message:
Amount total: [N]
Success: [N]
Fail: [N]
```

**No eligible transactions:**
```
Date: ddmmyyyy hh:mm:ss
File name: null
Message:
Amount total: 0
Success: 0
Fail: 0
```

---

## Acceptance Criteria

**Scenario 1: User triggers manual JV creation with valid filter**

Given eligible `Ungroup` transactions matching the selected company, date range, and event codes,
When the user confirms from AMS,
Then the system consolidates transactions, generates JV files (max 900 group docs/file) named `cccc_yyyymmdd_hhmmss_manual`, uploads to Manual S3 folder, updates included transactions' Grouping Status to `Grouped`, and sends Google Chat notification with all file names and result summary.

**Scenario 2: Multiple files when group count exceeds 900**

Given a manual request producing > 900 group documents,
When the system generates files,
Then multiple files are created with distinct sequential timestamp suffixes, each ≤ 900 group docs, all file names listed in notification.

**Scenario 3: No eligible transactions found**

Given a filter returning no `Ungroup` transactions,
When the user confirms,
Then no JV transaction or file is created; Google Chat notification sent with `File name: null` and all counts as zero.

**Scenario 4: Generated files are downloadable from AMS**

Given one or more JV files successfully created,
When user navigates to the JV display screen in AMS,
Then all generated files are listed with download links.

**Scenario 5: Transactions marked Grouped after file generation**

Given transactions included in a manual JV run,
When JV file generation completes,
Then all included transactions have Grouping Status = `Grouped` and are excluded from future auto batch and manual JV runs.

---

## Dependencies

| Dependency | Type |
|-----------|------|
| F14 (Outbound) — `sap_transaction` records with `Ungroup` status | Blocking |
| F15 (Auto Batch) — shares same grouping logic | Related |
| F16 (SAP Result) — result upload applies to both auto and manual JV files | Downstream |
