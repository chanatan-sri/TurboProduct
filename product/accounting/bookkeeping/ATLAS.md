# Bookkeeping — ATLAS

> Consolidated capability document. This is the **living, current-state truth** for this product.
> Source: bookkeeping_overview.md (Project 1020 — Release 1)
> Last Updated: 2026-03-04

---

## Overview

**Bookkeeping** is an internal accounting platform designed to centralise, validate, and automate the recording of accounting transactions across NTB business units. It serves as the single source of truth for double-entry ledger records, and acts as the integration layer between upstream business systems (AMS, LOS, Cash Reconcile) and the downstream financial reporting system (SAP FI).

**Project Code:** 1020
**Release:** Release 1 — Core Foundation

---

## System Architecture: Five Functional Domains

| Domain | Key Entities | Role |
|--------|-------------|------|
| **COA Management** | `master_fs_group`, `master_general_ledger`, `master_sub_ledger_type`, `master_tracking_dimension` | Configure Chart of Accounts — FS Groups, GL accounts, cost/profit centers, sub-ledger types |
| **Accounting Gateway** | `master_accounting_event`, `accounting_config_event`, `accounting_config_field_requirements`, `accounting_event_references` | Inbound intake — validate and route transaction requests to the Accounting Book |
| **Accounting Book** | `journal_header`, `journal_line`, `journal_line_attributes`, `journal_header_references` | Raw journal store — double-entry records with full reference data |
| **SAP Connector** | `sap_transaction`, `sap_jv_transaction`, `sap_jv_transaction_detail`, `sap_file`, `sap_config_jv_transaction` | Outbound SAP FI integration — JV batch grouping, file generation, result processing |
| **Master Data** | `master_entity`, `master_reference_field`, `master_accounting_type`, `master_sub_ledger_type` | Organisational reference tables supporting all domains |

**Supporting views:**
- `pivot_view` + `pivot_view_refresh_log` — async Book of Record (summary view)
- `log_request` + `log_response` — internal API audit logs

---

## Capabilities

### 1. COA Management
Manages the full Chart of Accounts hierarchy:
- **FS Groups** — financial statement groupings (Assets, Liabilities, SE, PL, OCI). 114 groups across 4 active types. API-only in Release 1.
- **General Ledger Accounts** — 10-digit numeric format (type · major group · sub-group · item). Each GL has sub-ledger type (Customer / Vendor / None) and center type (Profit / Cost).
- **Sub-Ledger Type Assignment** — designates whether a GL account requires a counterparty code.
- **Cost & Profit Center** — organizational units for transaction allocation. Real-time sync from upstream (F6).

### 2. Accounting Gateway
Two entry paths, both converging at the Accounting Book:

| Path | Caller | GL Resolution |
|------|--------|--------------|
| Upstream (F8) | LOS / Cash Reconcile via S3 | No GL in file — resolved from Event Config |
| Manual AMS (F10) | Accounting User via AMS | GL supplied by user — validated against COA |

- **Accounting Event Config (F5):** Each event code maps to DR/CR GL accounts, cost/profit center, and vendor/customer code. Gateway resolves GL from this config for upstream path.
- **Single Transaction API (F7):** POST endpoint for real-time single transaction submission.
- **Validation:** 2-stage — Stage 1 (file format, all-or-nothing), Stage 2 (row-level, valid rows continue).

### 3. Accounting Book
- **Raw Journal Store (F12):** Stores every validated double-entry record: `journal_header` + `journal_line` + `journal_line_attributes` + `journal_header_references`. Includes posting dates, active status flags.
- **Book of Record / Pivot View (F13):** Asynchronous summary view (`pivot_view`). Triggered by new transaction save and SAP result update.
- **Cancel Transaction (F19):** Marks a transaction as inactive. Only available before transaction is grouped into a JV batch.

### 4. SAP Connector
- **Outbound (F14):** Consolidates transactions with `Ungroup` grouping status into JV batches.
- **JV Export Auto Batch (F15):** Groups up to 900 document groups per JV file. Uploads to S3.
- **Manual JV File (F17):** Manual override to create a JV file outside the auto-batch cycle.
- **Upload SAP Result (F16):** Accounting user uploads SAP FI result file via AMS. System updates JV Posting Status (SUCCESS / FAILED) and propagates to linked transactions.
- **Cancel JV (F18):** Cancels a FAILED JV batch. Releases linked transactions back to `Ungroup` for re-grouping.

### 5. Master Data
- Entity registry (`master_entity`)
- Reference field definitions (`master_reference_field`)
- Accounting type catalogue (`master_accounting_type`)
- Sub-ledger type catalogue (`master_sub_ledger_type`)

---

## Transaction Status Model

### Grouping Status
| Status | Meaning |
|--------|---------|
| `Ungroup` | Transaction saved, eligible for JV batching |
| `Grouped` | Locked into a JV batch |
| `Do Not Post` | Permanently excluded (SAP Record flag = N, e.g., migration entries) |

### JV Posting Status
| Status | Meaning |
|--------|---------|
| `Ungroup` | Not yet batched |
| `PENDING` | In a JV batch, waiting for SAP result |
| `SUCCESS` | SAP FI confirmed posting |
| `FAILED` | SAP FI rejected — JV must be cancelled and re-batched |
| `Do Not Post` | Excluded from SAP permanently |

### JV Voucher Status
| Status | Meaning |
|--------|---------|
| `CONFIRMED` | JV batch created and file generated |
| `CANCELLED` | JV batch cancelled after FAILED result |

---

## Key Business Rules

| Rule | Detail |
|------|--------|
| Double-entry enforcement | Every transaction must have exactly one DR line and one CR line that balance |
| SAP Record flag | `Y` = eligible for JV batching; `N` = permanently excluded (Do Not Post) |
| JV file size limit | Max 900 document groups per JV file |
| Upstream path GL resolution | GL accounts are NEVER in the upstream file — resolved from Event Config only |
| Manual path GL resolution | GL accounts are supplied by user and validated against COA master data |
| Retired FS Group / GL | Inactive FS Groups and GL accounts cannot be referenced by new transactions |
| Failed JV recovery | Cancel JV → releases transactions to Ungroup → re-batch into new JV |
| Pivot view consistency | Pivot view is eventually consistent — async refresh, not real-time |

---

## User Flows

### End-to-End Transaction Flow
See [bookkeeping_overview.md §2](../../../bookkeeping_overview.md) for the full Mermaid diagram.

Key summary:
1. Upstream file or AMS user submits transaction
2. Stage 1: file format validation (all-or-nothing)
3. Stage 2: row-level validation (valid rows continue, failed rows notified)
4. Event Config Lookup (upstream path) or COA validation (manual path)
5. SAP Record flag check → Save to Accounting Book
6. Async pivot view refresh
7. Auto-batch Ungroup transactions → JV file → S3
8. Accounting user uploads to SAP FI manually
9. SAP result file uploaded → statuses updated → pivot view refresh

---

## Feature Index

| ID | Feature Name | Capability | Priority | Status |
|----|-------------|-----------|---------|--------|
| F1 | COA Management — FS Group Create & Update | COA Management | P1 | 📝 Spec |
| F2 | COA Management — General Ledger Create & Update | COA Management | P1 | 📝 Spec |
| F3 | COA Management — GL Sub-Ledger Type Assignment | COA Management | P2 | 📝 Spec |
| F4 | COA Management — Cost & Profit Center Create & Update | COA Management | P1 | 📝 Spec |
| F5 | Accounting Gateway — Accounting Event Template Config | Accounting Gateway | P1 | 📝 Spec |
| F6 | COA Management — Real-Time Cost & Profit Center Sync | COA Management | P1 | 📝 Spec |
| F7 | Accounting Gateway — Create Single Transaction (API) | Accounting Gateway | P1 | 📝 Spec |
| F8 | Accounting Gateway — Create Transactions by File Upload (Upstream) | Accounting Gateway | P1 | 📝 Spec |
| F10 | Accounting Gateway — Manual Create Transaction by File Upload (AMS) | Accounting Gateway | P1 | 📝 Spec |
| F12 | Accounting Book — Transaction Journal Record (Raw) | Accounting Book | P1 | 📝 Spec |
| F13 | Accounting Book — Book of Record (Summary / Pivot View) | Accounting Book | P2 | 📝 Spec |
| F14 | SAP Connector — Accounting Transaction Outbound | SAP Connector | P1 | 📝 Spec |
| F15 | SAP Connector — Journal Voucher Export (Auto Batch) | SAP Connector | P2 | 📝 Spec |
| F16 | SAP Integration — Upload SAP Transfer Result | SAP Connector | P2 | 📝 Spec |
| F17 | SAP Connector — Manual Create JV File | SAP Connector | TBC | 📝 Spec |
| F18 | SAP Connector — Cancel JV Transaction | SAP Connector | TBC | 📝 Spec |
| F19 | Cancel Accounting Transaction | Accounting Book | TBC | 📝 Spec |

---

## Changelog History

| # | Changelog | Summary | Date |
|---|-----------|---------|------|
| 001 | [CHANGELOG_001_initial.md](changelogs/CHANGELOG_001_initial.md) | Initial product definition — Accounting portfolio and Bookkeeping product onboarded into TurboProduct | 2026-03-04 |
