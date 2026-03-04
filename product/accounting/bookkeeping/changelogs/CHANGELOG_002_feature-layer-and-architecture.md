# CHANGELOG 002 — Feature Layer & Architecture

**Date**: 2026-03-04
**Layer**: Feature + Product (Architecture)
**Author**: Phasathon & Pojchara
**Status**: Final

---

## What Changed

### Feature Layer — 17 Feature Files Created

All Release 1 features sourced from `bookkeeping_overview.md` have been documented as individual `FEATURE_*.md` files.

#### COA Management (5 features)

| ID | Feature | File |
|----|---------|------|
| F1 | FS Group Create & Update | `capabilities/coa-management/features/FEATURE_F1_fs-group-create-update.md` |
| F2 | General Ledger Create & Update | `capabilities/coa-management/features/FEATURE_F2_gl-create-update.md` |
| F3 | GL Sub-Ledger Type Assignment | `capabilities/coa-management/features/FEATURE_F3_gl-sub-ledger-type-assignment.md` |
| F4 | Cost & Profit Center Create & Update | `capabilities/coa-management/features/FEATURE_F4_cost-profit-center-create-update.md` |
| F6 | Real-Time Cost & Profit Center Sync | `capabilities/coa-management/features/FEATURE_F6_realtime-center-sync.md` |

#### Accounting Gateway (4 features)

| ID | Feature | File |
|----|---------|------|
| F5 | Accounting Event Template Config | `capabilities/accounting-gateway/features/FEATURE_F5_accounting-event-template-config.md` |
| F7 | Create Single Accounting Transaction (API) | `capabilities/accounting-gateway/features/FEATURE_F7_create-single-transaction-api.md` |
| F8 | Create Transactions by File Upload (Upstream) | `capabilities/accounting-gateway/features/FEATURE_F8_create-transactions-file-upload.md` |
| F10 | Manual Create Transaction by File Upload (AMS) | `capabilities/accounting-gateway/features/FEATURE_F10_manual-create-transaction-file-upload.md` |

#### Accounting Book (3 features)

| ID | Feature | File |
|----|---------|------|
| F12 | Transaction Journal Record (Raw) | `capabilities/accounting-book/features/FEATURE_F12_transaction-journal-record.md` |
| F13 | Book of Record (Summary / Pivot View) | `capabilities/accounting-book/features/FEATURE_F13_book-of-record-pivot-view.md` |
| F19 | Cancel Accounting Transaction | `capabilities/accounting-book/features/FEATURE_F19_cancel-accounting-transaction.md` |

#### SAP Connector (5 features)

| ID | Feature | File |
|----|---------|------|
| F14 | Accounting Transaction Outbound | `capabilities/sap-connector/features/FEATURE_F14_accounting-transaction-outbound.md` |
| F15 | Journal Voucher Export (Auto Batch) | `capabilities/sap-connector/features/FEATURE_F15_jv-export-auto-batch.md` |
| F16 | Upload SAP Transfer Result | `capabilities/sap-connector/features/FEATURE_F16_upload-sap-transfer-result.md` |
| F17 | Manual Create JV File (AMS) | `capabilities/sap-connector/features/FEATURE_F17_manual-create-jv-file.md` |
| F18 | Cancel JV Transaction | `capabilities/sap-connector/features/FEATURE_F18_cancel-jv-transaction.md` |

> **Note on F9 and F11:** These IDs are intentionally skipped — they are gap IDs in the source document and have no corresponding feature in Release 1.

> **Note on Master Data:** Master Data features (Entity Registry, Reference Field Definitions, Accounting Type Catalogue, Sub-Ledger Type Catalogue) are managed as seed data in Release 1. No F-numbers assigned. F-numbers will be introduced in a future changelog if a self-service management UI is added.

---

### Product Layer — ARCHITECTURE.md Created

- Created `ARCHITECTURE.md` covering:
  - System overview diagram
  - Full data models for all 5 domains (~15 database entity tables with field-level detail)
  - Complete 43-column SAP JV file format specification
  - Infrastructure: S3 paths, Google Chat notification format, batch schedule, API base URL
  - Integration map (all inbound/outbound connections)
  - API Contracts: F6 endpoint documented; all others marked Pending (awaiting dev team spec)
  - 7 Architecture Decision Records (ADR-001 through ADR-007)

---

## Actions Closed from CHANGELOG_001

| Action | Status |
|--------|--------|
| Create FEATURE_<name>.md files for each P1 feature (F1–F8, F10, F12–F16) | ✅ Done — all 17 Release 1 features documented |
| Create ARCHITECTURE.md (data models, SAP JV schema, API contracts) | ✅ Done — API contracts section pending dev spec |

---

## Remaining Open Items

| # | Action | Layer | Owner |
|---|--------|-------|-------|
| 1 | Resolve open questions in each CAPABILITY.md | Capability | Phasathon & Pojchara |
| 2 | Complete API Contracts in ARCHITECTURE.md | Product | Engineering Owner |
| 3 | Clarify DaVinci dependency for `master_entity` | Product | Platform PO |
| 4 | Assign F-numbers to Master Data features (if self-service UI added in future release) | Capability | Phasathon & Pojchara |
