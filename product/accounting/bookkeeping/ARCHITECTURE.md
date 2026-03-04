# Bookkeeping — Architecture

> Technical design document. Defines the **"How"** of the Bookkeeping system.
> Source: bookkeeping_overview.md (Project 1020 — Release 1)
> Last Updated: 2026-03-04

---

## 1. System Overview

The Bookkeeping system is organized into **five functional domains**, each with its own set of database entities. All domains share a single database (RDS).

```
┌─────────────────────────────────────────────────────────────────┐
│                        BOOKKEEPING SYSTEM                       │
│                                                                 │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────────┐  │
│  │ COA          │    │  Accounting  │    │  Accounting      │  │
│  │ Management   │───▶│  Gateway     │───▶│  Book            │  │
│  │              │    │              │    │                  │  │
│  │ master_fs_   │    │ master_      │    │ journal_header   │  │
│  │ group        │    │ accounting_  │    │ journal_line     │  │
│  │ master_gl    │    │ event        │    │ journal_line_    │  │
│  │ master_org_  │    │ accounting_  │    │ attributes       │  │
│  │ unit         │    │ config_event │    │ journal_header_  │  │
│  └──────────────┘    └──────────────┘    │ references       │  │
│                                          └────────┬─────────┘  │
│  ┌──────────────┐    ┌──────────────┐             │            │
│  │  Master      │    │  SAP         │◀────────────┘            │
│  │  Data        │    │  Connector   │                          │
│  │              │    │              │                          │
│  │ master_      │    │ sap_         │                          │
│  │ entity       │    │ transaction  │                          │
│  │ master_      │    │ sap_jv_      │                          │
│  │ reference_   │    │ transaction  │                          │
│  │ field        │    │ sap_file     │                          │
│  └──────────────┘    └──────────────┘                          │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Data Models

### 2.1 COA Management Domain

#### `master_fs_group`
Financial Statement Group — top-level COA classification.

| Field | Type | Notes |
|-------|------|-------|
| id | INT | PK |
| fs_group_type_id | INT | FK → `master_fs_group_type` |
| fs_group_code | VARCHAR | Unique. Format: type prefix + numeric suffix (e.g., `A101`, `PL219B`, `OCI101`) |
| fs_group_name_th | VARCHAR | Thai name |
| fs_group_name_en | VARCHAR | English name |
| is_active | BOOLEAN | Default true. Inactive groups cannot be referenced by new GL accounts. |
| created_at | TIMESTAMP | |
| updated_at | TIMESTAMP | |

#### `master_fs_group_type`
Fixed catalogue — 6 types.

| ID | Type Code | Name EN |
|----|-----------|---------|
| 1 | `A` | Assets |
| 2 | `L` | Liabilities |
| 3 | `SE` | Shareholders' Equity |
| 4 | `R` | Revenue (no groups assigned in Release 1) |
| 5 | `E` | Expense (no groups assigned in Release 1) |
| 6 | `PL` | Profit & Loss (incl. OCI — uses `OCI` prefix convention) |

#### `master_general_ledger`
GL Account master.

| Field | Type | Notes |
|-------|------|-------|
| id | INT | PK |
| account_number | CHAR(10) | Unique. 10-digit numeric. Format: `[type 1][major 3][subgroup 2][item 4]` |
| account_name_th | VARCHAR | |
| account_name_en | VARCHAR | |
| fs_group_id | INT | FK → `master_fs_group`. Must be active. |
| sub_ledger_type_id | INT | FK → `master_sub_ledger_type`. (`Customer` / `Vendor` / `None`) |
| center_type | ENUM | `PROFIT_CENTER` or `COST_CENTER` — determines how center value is stored on journal lines |
| is_active | BOOLEAN | Default true |
| created_at | TIMESTAMP | |
| updated_at | TIMESTAMP | |

**GL Account Number Format:**
```
X  ·  XXX  ·  XX  ·  XXXX
│     │       │     └── Item (4 digits)
│     │       └──────── Sub-group (2 digits)
│     └──────────────── Major group (3 digits)
└────────────────────── Account type (1 digit): 1=Asset, 2=Liability, 4=Revenue, 5=Expense, 9=Suspense
```

#### `master_sub_ledger_type`
Fixed catalogue — 3 values.

| ID | Code | Meaning |
|----|------|---------|
| 1 | `Vendor` | Posting requires a vendor counterparty code |
| 2 | `Customer` | Posting requires a customer counterparty code |
| 3 | `None` | No counterparty required |

#### `master_organizational_unit`
Cost & Profit Center (branch/unit) master.

| Field | Type | Notes |
|-------|------|-------|
| id | INT | PK |
| unit_code | VARCHAR | Unique. Branch code (e.g., `NBI001`, `UATTEST01`) |
| unit_name | VARCHAR | Display name |
| unit_type_id | INT | FK → `master_unit_type` |
| is_active | BOOLEAN | |
| created_at | TIMESTAMP | |
| updated_at | TIMESTAMP | |

#### `master_unit_center_detail`
Center assignments per unit (one unit can have both cost and profit center).

| Field | Type | Notes |
|-------|------|-------|
| id | INT | PK |
| unit_id | INT | FK → `master_organizational_unit` |
| center_type_id | INT | `1` = Cost Center, `2` = Profit Center |
| center_code | VARCHAR | The actual center code (e.g., `1005111000`) |

---

### 2.2 Accounting Gateway Domain

#### `master_accounting_event`
Event catalogue — each event code has one entry.

| Field | Type | Notes |
|-------|------|-------|
| id | INT | PK |
| event_code | VARCHAR(7) | Unique. Format: `[Company 1][Category 3][Number 3]` (e.g., `BCOH001`) |
| event_name | VARCHAR | |
| company_entity_id | INT | FK → `master_entity` |
| sap_record_flag | BOOLEAN | `true` = eligible for JV batching; `false` = Do Not Post |
| is_active | BOOLEAN | |

#### `accounting_config_event`
GL mapping per event — DR and CR account resolution rules.

| Field | Type | Notes |
|-------|------|-------|
| id | INT | PK |
| accounting_event_id | INT | FK → `master_accounting_event` |
| dr_gl_account_id | INT | FK → `master_general_ledger` |
| cr_gl_account_id | INT | FK → `master_general_ledger` |
| dr_center_logic | ENUM | `BRANCH_FROM_REF1` / `FIXED_HQ` / `FIXED_AM` |
| cr_center_logic | ENUM | `BRANCH_FROM_REF1` / `FIXED_HQ` / `FIXED_AM` |
| dr_fixed_center_code | VARCHAR | Populated when `dr_center_logic = FIXED_HQ` or `FIXED_AM` |
| cr_fixed_center_code | VARCHAR | Populated when `cr_center_logic = FIXED_HQ` or `FIXED_AM` |
| dr_sub_ledger_code | VARCHAR | Fixed vendor/customer code for DR line (e.g., `1020`) — null if not required |
| cr_sub_ledger_code | VARCHAR | Fixed vendor/customer code for CR line — null if not required |
| doc_date_logic | ENUM | `TRANSACTION_DATE` / `STATEMENT_DATE` / `MANUAL` |

#### `accounting_config_field_requirements`
Reference field requirements per event.

| Field | Type | Notes |
|-------|------|-------|
| id | INT | PK |
| accounting_event_id | INT | FK → `master_accounting_event` |
| reference_field_id | INT | FK → `master_reference_field` (Ref1–Ref6) |
| is_required | BOOLEAN | `true` = mandatory, `false` = optional |

---

### 2.3 Accounting Book Domain

#### `journal_header`
One record per accounting transaction.

| Field | Type | Notes |
|-------|------|-------|
| id | INT | PK |
| accounting_event_id | INT | FK → `master_accounting_event` |
| entity_id | INT | FK → `master_entity` |
| posting_date | DATE | |
| doc_date | DATE | |
| data_integrity_status | ENUM | `Active` / `Void` — set to `Void` by F19 Cancel |
| created_by | INT | User or system that created the transaction |
| created_at | TIMESTAMP | |

#### `journal_line`
Two records per transaction (DR + CR).

| Field | Type | Notes |
|-------|------|-------|
| id | INT | PK |
| journal_header_id | INT | FK → `journal_header` |
| line_type | ENUM | `DR` / `CR` |
| gl_account_id | INT | FK → `master_general_ledger` |
| accounting_type_id | INT | FK → `master_accounting_type` |
| amount | DECIMAL(15,2) | Always positive — line_type determines DR/CR direction |

#### `journal_line_attributes`
Optional attributes per journal line (cost/profit center, sub-ledger).

| Field | Type | Notes |
|-------|------|-------|
| id | INT | PK |
| journal_line_id | INT | FK → `journal_line` |
| attribute_type | ENUM | `COST_CENTER` / `PROFIT_CENTER` / `VENDOR_CODE` / `CUSTOMER_CODE` |
| attribute_value | VARCHAR | The actual center code or counterparty code |

#### `journal_header_references`
Reference1–Reference6 values per transaction.

| Field | Type | Notes |
|-------|------|-------|
| id | INT | PK |
| journal_header_id | INT | FK → `journal_header` |
| reference_field_id | INT | FK → `master_reference_field` (1–6) |
| reference_value | VARCHAR | |

#### `pivot_view`
Async summary view — eventually consistent with `journal_header`.

| Field | Type | Notes |
|-------|------|-------|
| id | INT | PK |
| journal_header_id | INT | FK → `journal_header` |
| company_code | VARCHAR | |
| event_code | VARCHAR | |
| posting_date | DATE | |
| dr_account_number | VARCHAR | |
| cr_account_number | VARCHAR | |
| amount | DECIMAL(15,2) | |
| reference_1–6 | VARCHAR | Flattened reference values |
| sap_group_doc | VARCHAR | JV Doc Header Text (from SAP Connector) |
| grouping_status | VARCHAR | Current grouping lifecycle status |
| posting_status | VARCHAR | Current inherited posting status |
| updated_at | TIMESTAMP | Last refresh timestamp |

#### `pivot_view_refresh_log`
Audit log of every pivot view refresh.

| Field | Type | Notes |
|-------|------|-------|
| id | INT | PK |
| refresh_type | ENUM | `TRANSACTION_SAVE` / `SAP_RESULT_UPDATE` |
| status | ENUM | `SUCCESS` / `ERROR` |
| records_processed | INT | |
| error_message | VARCHAR | Null on success |
| triggered_at | TIMESTAMP | |

---

### 2.4 SAP Connector Domain

#### `sap_transaction`
SAP tracking record — one per `journal_header`.

| Field | Type | Notes |
|-------|------|-------|
| id | INT | PK |
| journal_header_id | INT | FK → `journal_header` — 1:1 |
| grouping_lifecycle_status | ENUM | `Ungroup` / `Grouped` / `Do Not Post` |
| inherited_posting_status | ENUM | `Ungroup` / `Submitted` / `Posted` / `Failed` / `Do Not Post` |
| sap_jv_transaction_id | INT | FK → `sap_jv_transaction` — null until grouped |
| updated_at | TIMESTAMP | |

#### `sap_transaction_history`
Append-only status change log.

| Field | Type | Notes |
|-------|------|-------|
| id | INT | PK |
| sap_transaction_id | INT | FK → `sap_transaction` |
| previous_grouping_status | VARCHAR | |
| previous_posting_status | VARCHAR | |
| changed_at | TIMESTAMP | |

#### `sap_jv_transaction`
One JV batch = one group document in the JV file.

| Field | Type | Notes |
|-------|------|-------|
| id | INT | PK |
| doc_header_text | VARCHAR | Format: `YYYYMMDDEventCode_000000NNN` — primary key for SAP result matching |
| company_code | VARCHAR(4) | `1000` NTB / `1010` NTBPL / `1020` NTBI / `1030` NTBX |
| event_code | VARCHAR | |
| posting_date | DATE | |
| doc_date | DATE | |
| voucher_status | ENUM | `CONFIRMED` / `CANCELLED` |
| posting_status | ENUM | `PENDING` / `SUCCESS` / `FAILED` |
| sap_file_id | INT | FK → `sap_file` |
| grp_doc_number | INT | Sequential group doc number within the file |
| created_at | TIMESTAMP | |

#### `sap_file`
One record per generated JV file.

| Field | Type | Notes |
|-------|------|-------|
| id | INT | PK |
| file_name | VARCHAR | Auto: `NTBxxx_yyyymmdd`. Manual: `cccc_yyyymmdd_hhmmss_manual` |
| file_type | ENUM | `AUTO` / `MANUAL` |
| s3_path | VARCHAR | Full S3 path |
| total_group_docs | INT | Number of group documents in the file (max 900) |
| created_at | TIMESTAMP | |

---

### 2.5 Master Data Domain

#### `master_entity`
NTB legal entities.

| Field | Type | Notes |
|-------|------|-------|
| id | INT | PK |
| company_code | VARCHAR(4) | `1000`, `1010`, `1020`, `1030` |
| entity_name | VARCHAR | |
| is_active | BOOLEAN | |

#### `master_reference_field`
Defines Reference1–Reference6 metadata.

| Field | Type | Notes |
|-------|------|-------|
| id | INT | PK (1–6) |
| field_name | VARCHAR | e.g., `Reference1` |
| field_description | VARCHAR | e.g., `Branch code (รหัสสาขา)` |

#### `log_request` / `log_response`
API audit logs — created for every inbound request.

| Field | Type | Notes |
|-------|------|-------|
| id | INT | PK |
| endpoint | VARCHAR | |
| payload | JSON | |
| status | VARCHAR | |
| created_at | TIMESTAMP | |

---

## 3. SAP JV File Format

The JV upload file is a flat file with **43 columns**, sent to SAP FI.

**Naming Convention:**
- Auto batch: `NTBxxx_yyyymmdd` (e.g., `NTB001_20250814`)
- Manual: `cccc_yyyymmdd_hhmmss_manual` (e.g., `1000_20251024_100143_manual`)

**Constraints:**
- Max **900 group documents** per file — overflow creates a new file
- Each group document = exactly **2 rows** (1 DR + 1 CR)
- All amounts within a group must **net to zero**

### Column Reference

| # | Column | Length | Required | Rules |
|---|--------|--------|----------|-------|
| 1 | Grp Doc | 3 | Yes | Sequential per file. Resets to 1 when new file is created (> 900 overflow). |
| 2 | Company Code | 4 | Yes | `1000`=NTB, `1010`=NTBPL, `1020`=NTBI, `1030`=NTBX |
| 3 | Ledger | 2 | No | Blank for standard entries |
| 4 | Posting Date | 8 | Yes | Format `ddmmyyyy` — no separator |
| 5 | Document Date | 8 | Yes | Format `ddmmyyyy` — no separator |
| 6 | Doc Type | 2 | Yes | `SA` = HQ entries, `PT` = branch entries |
| 7 | Currency | 3 | Yes | Always `THB` |
| 8 | Exchange Rate | (4,5) | No | Blank for THB transactions |
| 9 | Reference | 16 | Yes | Accounting Event Code (e.g., `BOPE001`) |
| 10 | Doc Header Text | 25 | Yes | `YYYYMMDDEventCode_000000NNN` — **primary key for SAP result matching** |
| 11 | Business Place / Value Date | 4 | Yes | Always `0` |
| 12 | BP Branch Code (BCODE) | 5 | No | Branch code |
| 13 | BP Tax ID (TAXID) | 13 | No | Tax ID |
| 14 | Posting Key | 2 | Yes | See Posting Key Rules below |
| 15 | Account | 7 | Yes | GL account number — or Customer/Vendor number if sub-ledger applies |
| 16 | Special G/L Indicator | 1 | No | — |
| 17 | Amount | (13,2) | Yes | Positive for DR keys (40, 01, 21). Negative for CR keys (50, 31, 11). Lines in group must sum to zero. |
| 18 | Tax Code | 2 | No | — |
| 19 | Tax Base Amount | (13,2) | No | — |
| 20 | Payment Term | 4 | No | — |
| 21 | Baseline Date / Value Date | 8 | No | Format `ddmmyyyy` |
| 22 | Payment Method | 1 | No | — |
| 23 | Cost Center | 10 | No | From DR/CR cost center on the accounting transaction |
| 24 | Profit Center | 10 | No | From DR/CR profit center on the accounting transaction |
| 25 | Order | 12 | No | — |
| 26 | Assignment | 18 | No | — |
| 27 | Text | 50 | No | Transaction description / remark |
| 28–31 | WHT Type/Code/Base/Amount 1 | Various | No | Withholding tax set 1 |
| 32–35 | WHT Type/Code/Base/Amount 2 | Various | No | Withholding tax set 2 |
| 36–43 | Name1, Name2, Street, City, Postal Code, Country, Bank Key, Bank Account | Various | No | Vendor/Customer address and banking |

### Posting Key Rules

| Account Type | DR Key | CR Key |
|---|---|---|
| Customer | 01 | 11 |
| Vendor | 21 | 31 |
| General Ledger | 40 | 50 |

### Grouping Attributes

Transactions are consolidated into one group document when ALL of the following match:
Company Code, Posting Date, Document Date, Doc Type, Currency, Reference (Event Code), Business Place, Posting Key, Account, Baseline Date, Profit Center, Text.

### Example — Raw Transactions → JV File

```
Source transactions (Event BPAY001, Company 1000):
  Case 6:  DR 1110000000  +100,000  |  CR 9111000021  -100,000
  Case 7:  DR 1110000000   +55,000  |  CR 9111000021   -55,000
  Case 8:  DR 1110000000  +350,530  |  CR 9111000021  -350,530

JV File output — Group Doc 4 (2 rows, nets to zero):
  Row 1: Grp Doc 4 | Co 1000 | 27082025 | SA | THB | BPAY001 | 20250828BPAY001_000000001
         PK 40  Account 1110000000  Amount +505,530.00  (DR)
  Row 2: Grp Doc 4 | Co 1000 | 27082025 | SA | THB | BPAY001 | 20250828BPAY001_000000001
         PK 50  Account 9111000021  Amount -505,530.00  (CR)
```

---

## 4. Infrastructure

### S3 Storage Paths

| Purpose | S3 Path |
|---------|---------|
| JV Auto Batch files | `daily-report/Accounting/JVupload/Bookkeeping/auto/yyyy/mm/NTBxxx_yyyymmdd` |
| JV Manual files | `daily-report/Accounting/JVupload/Bookkeeping/manual/cccc_yyyymmdd_hhmmss_manual` |
| Upstream transaction files (F8) | Upstream system uploads directly to S3 |
| Processing result files (F8, F10, F16) | Linked in Google Chat notification |

### Notification Channel

All processing notifications (F8, F10, F15, F16, F17) are sent to **Google Chat**.

### Batch Schedule

| Job | Schedule | Notes |
|-----|----------|-------|
| JV Auto Batch (F15) | T+1 nightly | Exact time TBC — pending dev team confirmation |
| Pivot View Refresh (F13) | Async — event-driven | Triggered by: (1) new transaction save, (2) SAP result update |

### API Base URL (UAT)

`https://bks-api.uat01.ntbx.tech`

---

## 5. Integration Map

```
INBOUND
  AMS          → POST /api/bookkeeping/transactions        (F7 — single transaction)
  AMS          → File upload via AMS screen                (F10 — manual bulk)
  LOS / Cash   → S3 file upload                           (F8 — upstream bulk)
  LOS          → POST /api/bookkeeping/organization-units  (F6 — branch sync)
  AMS          → File upload via AMS screen                (F16 — SAP result)

OUTBOUND
  Bookkeeping  → S3 JV file                               (F15 — auto batch)
  Bookkeeping  → S3 JV file                               (F17 — manual)
  Bookkeeping  → Google Chat notification                  (F8, F10, F15, F16, F17)
```

---

## 6. API Contracts

> **Status: Pending**
>
> The API specification document has not yet been provided by the dev team (Open Item #2 — bookkeeping_overview.md Section 7).
> This section will be updated when the spec is received.
>
> Known endpoint (documented): `POST /api/bookkeeping/organization-units` — see [FEATURE_F6](capabilities/coa-management/features/FEATURE_F6_realtime-center-sync.md) for full spec.
>
> Endpoints pending spec:
> - `POST /api/bookkeeping/transactions` (F7)
> - COA management endpoints — FS Group, GL Account, Center (F1, F2, F4)
> - Accounting Event config endpoint (F5)
> - Cancel transaction endpoint (F19)
> - Cancel JV batch endpoint (F18)

---

## 7. Architecture Decision Records (ADRs)

| # | Decision | Rationale |
|---|----------|-----------|
| ADR-001 | Pivot view is a separate RDS table, not a live query | Avoids performance impact of real-time joins across journal tables for accounting team queries |
| ADR-002 | Pivot view refresh is async (event-driven), not real-time | Acceptable for accounting use case — eventual consistency tolerated; real-time not required |
| ADR-003 | JV file max 900 group documents per file | SAP FI hard constraint — exceeding causes upload failure |
| ADR-004 | Upstream path (F8) does not include GL accounts in file | GL resolution is owned by Bookkeeping via Event Config — upstream systems should not need to know accounting logic |
| ADR-005 | SAP Record flag on each transaction | Allows migration entries and corrections to exist in Bookkeeping as records without being posted to SAP |
| ADR-006 | `Do Not Post` Posting Status preserved after JV cancellation | Historical record of SAP failure — enables audit trail of why a batch was cancelled |
| ADR-007 | COA management is API-only in Release 1 (no self-service UI) | Reduce scope for Release 1; accounting team changes are infrequent and go through dev team |
