# Bookkeeping System — Product Requirements Overview
**Project:** 1020 - Bookkeeping
**Document Type:** System Overview & Feature Summary (Release 1)
**Status:** Draft — Pending Technical Spec Integration
**Last Updated:** 2026-03-01

---

## Table of Contents

1. [Project Background](#1-project-background)
2. [System Architecture Overview](#2-system-architecture-overview)
   - [Accounting Transaction Flow](#accounting-transaction-flow)
3. [Domain Map](#3-domain-map)
4. [Release Scope](#4-release-scope)
5. [Feature Index](#5-feature-index)
6. [Feature Specifications](#6-feature-specifications)
7. [Open Items & Flags](#7-open-items--flags)

---

## 1. Project Background

The Bookkeeping system is an internal accounting platform designed to centralize, validate, and automate the recording of accounting transactions across NTB business units. It serves as the single source of truth for double-entry ledger records, and acts as the integration layer between upstream business systems (AMS, LOS, Cash Reconcile) and the downstream financial reporting system (SAP FI).

**Primary Objectives for Release 1:**

- Validate the core business concept and use cases for automated accounting event recording.
- Gather detailed accounting gateway usage patterns from the accounting team.
- Establish a configurable Chart of Accounts (COA) foundation — FS Groups, General Ledger accounts, Sub-ledgers, and Cost/Profit Centers.
- Implement an Accounting Gateway capable of receiving transaction requests via API and file upload.
- Build an Accounting Book (raw journal store) and a summary Book of Record (pivot view).
- Integrate with SAP FI via JV file export and result upload.

**Reference Documents:**

| Document | Link |
|---|---|
| BRD | BRD_Bookkeeping System |
| IT Blueprint | Figma Board |
| Business Concept (PO) | Overview concept link |
| Master Data Sheet | Google Sheets |
| Project Timeline | Monday.com Board |

---

## 2. System Architecture Overview

The system is organized into five functional domains, each reflected in the database schema.

### Domain Overview

**COA Management** handles the configuration of the Chart of Accounts, including FS Groups, General Ledger accounts, and Cost/Profit Centers. In Release 1, all COA management operations are performed via internal API (no UI for accounting team direct access). Sub-ledger type (Customer or Vendor) is an attribute set on each GL account at the time of creation — it indicates that transactions posted to that account require a counterparty code. The actual counterparty code value is not managed here; it is configured as a fixed value within the Accounting Event configuration (Feature F5).

**Accounting Gateway** is the inbound layer. It receives accounting transaction requests — either via API (single transaction) or file upload (bulk) — validates them against the configured Accounting Event template, and writes valid transactions to the Accounting Book.

**Accounting Book** is the raw journal store. It holds every validated double-entry record with full reference data, posting dates, and active status flags. A derived summary view (`pivot_view`) provides the Book of Record for reporting and querying.

**SAP Connector** is the outbound integration layer. It consolidates Accounting Transactions with `Ungroup` status into JV Transactions, generates JV upload files (max 900 document groups per file), posts them to S3, and processes inbound SAP posting result files to update transaction statuses.

**Organizational & Master Data** provides reference tables supporting all domains: entities, accounting event configs, reference field definitions, and unit/branch structures.

### Accounting Transaction Flow

The following diagram covers the complete end-to-end flow from transaction creation through to SAP posting result and pivot view refresh. There are two entry points, each with distinct file formats and validation logic.

```mermaid
flowchart TD
    classDef entryNode    fill:#3b2d8f,stroke:#7c6fcd,stroke-width:2px,color:#e8e0ff,font-weight:bold
    classDef userNode     fill:#3b2d8f,stroke:#7c6fcd,stroke-width:2px,color:#e8e0ff
    classDef validateNode fill:#0f2a45,stroke:#4a90d9,stroke-width:2px,color:#cce4ff
    classDef decisionNode fill:#2d2000,stroke:#c8960c,stroke-width:2px,color:#ffe082,font-weight:bold
    classDef notifyNode   fill:#3d1a00,stroke:#e07820,stroke-width:2px,color:#ffcc80,font-style:italic
    classDef configNode   fill:#0d3028,stroke:#2eb89a,stroke-width:2px,color:#80ffda
    classDef saveNode     fill:#0d2a00,stroke:#52c41a,stroke-width:2px,color:#b7eb8f,font-weight:bold
    classDef sapNode      fill:#2a1a3d,stroke:#9b6dff,stroke-width:2px,color:#d3b8ff
    classDef pivotNode    fill:#003030,stroke:#00bfbf,stroke-width:2px,color:#7fffee
    classDef endNode      fill:#1a1a1a,stroke:#555,stroke-width:1px,color:#888

    %% ─────────────────────────────────────────────────────
    %% ENTRY POINTS
    %% ─────────────────────────────────────────────────────
    A1(["☁  Upstream System
    Upload transaction file to S3
    ─────────────────────────────
    File contains: Event code · Posting date
    Doc date · Amount · Reference1-6 · Note
    GL accounts are NOT included"])

    A2(["👤  Accounting User · AMS
    Manual entry or file upload
    ─────────────────────────────
    File contains: Event code · Posting date · Doc date
    Dr/Cr Account Number · Dr/Cr Center
    Dr/Cr Sub-ledger · Amount · Reference1-6 · Remark"])

    %% ─────────────────────────────────────────────────────
    %% PATH 1 — UPSTREAM: STAGE 1
    %% ─────────────────────────────────────────────────────
    A1 --> S1["📋  Stage 1 · File Format Validation
    ─────────────────────────────────────
    File-level gate — all or nothing
    ─────────────────────────────────────
    · Column names match format
    · Required fields NOT NULL
    · Date format YYYY-MM-DD
    · Amount is numeric"]

    S1 --> D1{"Format
    valid?"}
    D1 -->|"✕  Invalid"| N1["🔔  Notify Dev Team
    File format error
    File halted — no rows processed"]
    N1 --> HALT(["⊘  End"])

    %% ─────────────────────────────────────────────────────
    %% PATH 1 — UPSTREAM: STAGE 2
    %% ─────────────────────────────────────────────────────
    D1 -->|"✓  Valid"| S2["🔍  Stage 2 · Row-Level Validation
    ──────────────────────────────────────────────
    Each row validated independently.
    No GL lookup — GL accounts are not in this file.
    All GL resolution happens in event config below.
    ──────────────────────────────────────────────
    ①  Event code exists in Bookkeeping
    ②  Amount > 0
    ③  Date fields conform to YYYY-MM-DD
    ④  Required reference fields NOT NULL"]

    S2 --> D2{"Any rows
    fail?"}
    D2 -->|"✕  Failed rows"| N2["🔔  Notify Dev Team
    Validation summary file
    ──────────────────────
    · ไม่พบ Event Code บนระบบ
    · amount มีค่าติดลบ
    · Format Date ไม่ถูกต้อง
    · กรุณาระบุ [Field]"]
    D2 -->|"✓  Passed rows"| CFG
    N2 -. "valid rows continue" .-> CFG

    %% ─────────────────────────────────────────────────────
    %% PATH 1 — EVENT CONFIG LOOKUP
    %% ─────────────────────────────────────────────────────
    CFG["⚙  Event Config Lookup
    ─────────────────────────────────────────────
    All GL resolution happens here from config.
    ─────────────────────────────────────────────
    ①  Event code → DR / CR GL accounts
    ②  Ref1 or fixed config → Profit / Cost Center
    ③  Event config → fixed Vendor / Customer code
    ④  Date logic per event rule"]

    %% ─────────────────────────────────────────────────────
    %% PATH 2 — MANUAL AMS: STAGE 1
    %% ─────────────────────────────────────────────────────
    A2 --> S1B["📋  Stage 1 · File Format Validation
    ─────────────────────────────────────
    File-level gate — all or nothing
    ─────────────────────────────────────
    · Column names match format
    · Required fields NOT NULL
    · Date format YYYY-MM-DD
    · Amount is numeric"]

    S1B --> D1B{"Format
    valid?"}
    D1B -->|"✕  Invalid"| N1B["🔔  Notify User via AMS
    File format error
    File halted — no rows processed"]
    N1B --> HALT2(["⊘  End"])

    %% ─────────────────────────────────────────────────────
    %% PATH 2 — MANUAL AMS: STAGE 2
    %% ─────────────────────────────────────────────────────
    D1B -->|"✓  Valid"| S2B["🔍  Stage 2 · Row-Level Master Data Validation
    ──────────────────────────────────────────────
    Each row validated independently.
    User supplies GL accounts directly — system
    validates each against COA and GL master config.
    ──────────────────────────────────────────────
    ①  Event code exists in Bookkeeping
    ②  DR Account Number exists in COA
    ③  CR Account Number exists in COA
    ④  DR Profit/Cost Center exists in COA
       GL master → determines Profit or Cost type
    ⑤  CR Profit/Cost Center exists in COA
       GL master → determines Profit or Cost type
    ⑥  DR Sub-ledger exists in COA
       GL master → Customer / Vendor / None
    ⑦  CR Sub-ledger exists in COA
       GL master → Customer / Vendor / None
    ⑧  Amount > 0
    ⑨  Date fields conform to YYYY-MM-DD"]

    S2B --> D2B{"Any rows
    fail?"}
    D2B -->|"✕  Failed rows"| N2B["🔔  Notify User via AMS
    Validation summary
    ──────────────────────────────────
    · ไม่พบ Event Code บนระบบ
    · ไม่พบ Dr/Cr Account บนระบบ
    · ไม่พบ Dr/Cr Center บนระบบ
    · ไม่พบ Dr/Cr Sub-ledger บนระบบ
    · amount มีค่าติดลบ
    · Format amount ไม่ถูกต้อง
    · Format Date ไม่ถูกต้อง
    · กรุณาระบุ [Field]"]
    D2B -->|"✓  Passed rows"| CFG2
    N2B -. "valid rows continue" .-> CFG2

    CFG2["✓  Values confirmed from user input
    Center type and sub-ledger type
    resolved from GL master lookup.
    No event config lookup performed."]

    %% ─────────────────────────────────────────────────────
    %% BOTH PATHS CONVERGE — SAP RECORD FLAG
    %% ─────────────────────────────────────────────────────
    CFG  --> SAPF{"SAP Record
    flag?"}
    CFG2 --> SAPF

    SAPF -->|"Y · Eligible for JV"| SAVE["💾  Save Accounting Transaction
    ─────────────────────────────────────
    Grouping Status   :  Ungroup
    Posting Status    :  Ungroup
    ─────────────────────────────────────
    journal_header · journal_line
    journal_line_attributes
    journal_header_references · sap_transaction"]

    SAPF -->|"N · Excluded from JV"| SAVED["💾  Save Accounting Transaction
    ─────────────────────────────────────
    Grouping Status   :  Do Not Post
    Posting Status    :  Do Not Post
    ─────────────────────────────────────
    Migration entries or corrections
    already in SAP — permanently excluded
    from JV batching"]

    SAVE  --> PV1
    SAVED --> PV1

    PV1["↺  Async · Pivot View Refresh
    Triggered by new transaction save
    pivot_view updated"]

    %% ─────────────────────────────────────────────────────
    %% JV BATCHING & SAP POSTING CYCLE
    %% ─────────────────────────────────────────────────────
    SAVE --> JV1["📦  Group Ungroup Transactions → JV Batch
    ──────────────────────────────────────────────
    Voucher Status   :  CONFIRMED
    Posting Status   :  PENDING
    Linked transactions Grouping → Grouped · locked"]

    JV1 --> JV2["📄  Generate JV File · Upload to S3
    Max 900 document groups per file"]

    JV2 --> JV3(["👤  Accounting User
    Download JV file from S3 via AMS
    Upload manually into SAP FI"])

    JV3 --> JV4["🏦  SAP FI
    Processes the upload
    Produces result file"]

    JV4 --> JV5(["👤  Accounting User
    Upload SAP result file via AMS
    Feature F16"])

    JV5 --> JV6["⚙  Process SAP Result
    ──────────────────────────────────────────────
    JV Posting Status         →  SUCCESS or FAILED
    Inherited Posting Status  →  propagated to each
                                 linked transaction"]

    JV6 --> D4{"Posting
    result?"}

    D4 -->|"✓  SUCCESS"| PV2["↺  Async · Pivot View Refresh
    Triggered by SAP result update
    pivot_view updated
    pivot_view_refresh_log recorded"]

    D4 -->|"✕  FAILED"| F18["🔄  Cancel JV Batch via AMS · F18
    ──────────────────────────────────────────────
    Voucher Status   :  CONFIRMED → CANCELLED
    Posting Status   :  FAILED  (retained as history)
    Linked transactions released
    Grouping Status  →  Ungroup
    Eligible for re-grouping into new JV batch"]

    F18 --> JV1

    %% ─────────────────────────────────────────────────────
    %% APPLY CLASSES
    %% ─────────────────────────────────────────────────────
    class A1,A2                    entryNode
    class JV3,JV5                  userNode
    class S1,S2,S1B,S2B            validateNode
    class D1,D2,D1B,D2B,SAPF,D4   decisionNode
    class N1,N2,N1B,N2B            notifyNode
    class CFG,CFG2                 configNode
    class SAVE,SAVED               saveNode
    class JV1,JV2,JV4,JV6,F18     sapNode
    class PV1,PV2                  pivotNode
    class HALT,HALT2               endNode
```

> **Upstream path (F8):** The file contains only Event code, dates, amount, and reference fields. No GL accounts, centers, or sub-ledger codes are submitted. Stage 2 validates only that the event code exists and the data fields are correctly formatted. All GL resolution — DR/CR accounts, cost/profit center, vendor/customer code — is performed automatically from the event config after validation passes.

> **Manual AMS path (F10):** The file includes DR/CR GL account numbers, center values, and sub-ledger codes supplied directly by the user. Stage 2 validates all of these against COA master data, and consults the GL master to determine whether each account requires a Profit Center or Cost Center and whether it carries a Customer, Vendor, or None sub-ledger type. No event config lookup is performed for GL resolution.

> **SAP Record flag (`Y` / `N`):** Each transaction row carries this flag. Rows flagged `N` are saved immediately with Grouping Status and Inherited Posting Status set to `Do Not Post`, permanently excluded from JV batching. This covers migration entries or manual corrections that already exist in SAP.

> **Pivot view refresh:** The refresh is asynchronous, triggered independently by two events — a new transaction save and a SAP result update.

> **JV batch failure recovery:** A FAILED JV batch can be cancelled via F18, which sets Voucher Status to `CANCELLED` and releases all linked transactions back to `Ungroup` for re-grouping. The FAILED Posting Status is retained as a historical record.llowing table maps each database entity group to its functional domain, based on the ERD schema.

| Domain | Key Entities | Color Code |
|---|---|---|
| SAP Integration & Transactions | `sap_transaction`, `sap_jv_transaction`, `sap_jv_transaction_detail`, `sap_file`, `sap_config_jv_transaction` | Blue |
| Journal & General Ledger | `journal_header`, `journal_line`, `journal_line_attributes`, `journal_header_references`, `master_general_ledger` | Orange |
| Accounting Configuration | `master_accounting_event`, `accounting_config_event`, `accounting_config_field_requirements`, `accounting_event_references` | Teal |
| Master Data & Types | `master_entity`, `master_reference_field`, `master_fs_group`, `master_accounting_type`, `master_sub_ledger_type` | Purple |
| Internal Logs & API | `log_request`, `log_response` | Red |
| Pivot View & Refresh | `pivot_view`, `pivot_view_refresh_log` | Green |

---

## 4. Release Scope

### Release 1 — Core Foundation (Current)

Release 1 establishes the core data model, configuration APIs, and the end-to-end flow from transaction request → journal record → SAP JV file. All COA management features are delivered as API tools only (no self-service UI for the accounting team).

### Release 2 — Parallel Run & Reconciliation

Release 2 will run the Bookkeeping system in parallel with the current system, using it primarily as a reconciliation tool to validate accuracy before full cutover.

### Release 3 — Full Cutover

Details to be confirmed.

---

## 5. Feature Index

| ID | POB Ticket | Feature Name | Type | Priority |
|---|---|---|---|---|
| F1 | POB-2197 | COA Management — FS Group Create & Update | New Feature | P1 |
| F2 | POB-2199 | COA Management — General Ledger Create & Update | New Feature | P1 |
| F3 | POB-2194 | COA Management — GL Sub-Ledger Type Assignment | New Feature | P2 |
| F4 | POB-2195 | COA Management — Cost & Profit Center Create & Update | New Feature | P1 |
| F5 | POB-2192 | Accounting Gateway — Accounting Event Template Config | New Feature | P1 |
| F6 | POB-2193 | COA Management — Real-Time Cost & Profit Center Sync from Upstream System | New Feature | P1 |
| F7 | POB-2205 | Accounting Gateway — Create Single Transaction (API) | New Feature | P1 |
| F8 | POB-2198 | Accounting Gateway — Create Transactions by File Upload | New Feature | P1 |
| F10 | POB-2206 | Accounting Gateway — Manual Create Transaction by File Upload | Enhancement | P1 |
| F12 | POB-2201 | Accounting Book — Transaction Journal Record (Raw) | New Feature | P1 |
| F13 | POB-2203 | Accounting Book — Book of Record (Summary / Pivot View) | New Feature | P2 |
| F14 | POB-2191 | SAP Connector — Accounting Transaction Outbound | New Feature | P1 |
| F15 | POB-2202 | SAP Connector — Journal Voucher Export (Auto Batch) | New Feature | P2 |
| F16 | POB-2624 | SAP Integration — Upload SAP Transfer Result | New Feature | P2 |
| F17 | POB-2363 | SAP Connector — Manual Create JV File | New Feature | TBC |
| F18 | POB-2389 | SAP Connector — Cancel JV Transaction | New Feature | TBC |
| F19 | POB-2417 | Cancel Accounting Transaction | New Feature | TBC |

---

## 6. Feature Specifications

---

### Feature F1: COA Management — FS Group Create & Update

**Type:** New Feature | **Domain:** Accounting / COA Management
**POB:** POB-2197 | **Priority:** P1

#### 1. Business Context & Value

As an accounting system administrator,
I want to create, update, and retire FS Groups via API,
So that General Ledger accounts can be correctly classified under the right financial statement groupings for reporting and compliance.

#### 2. System Context (The Delta Map)

**Current State (As-Is):** FS group configuration is maintained manually outside the system. There is no structured API or validation layer enforcing FS group integrity.

**Target State (To-Be):** The system exposes an API endpoint for creating and managing FS Groups. Each group is validated at the time of creation, and inactive groups cannot be referenced by new GL accounts.

**Components Impacted:** `master_fs_group`, `master_fs_group_type`, `master_general_ledger`

#### 3. Data Specification

**FS Group Type Catalogue**

The system defines six FS Group Types. Each type has a `type_code` which serves as the prefix anchor for FS Group codes belonging to that type. In Release 1, only types A, L, SE, and PL are actively used. Types R (Revenue) and E (Expense) are defined in the schema but have no groups assigned — all income and expense items are classified under type PL. OCI groups share the `PL` type (`fs_group_type_id = 6`) at the database level but use the `OCI` prefix convention in their group codes.

| ID | Type Code | Name (TH) | Name (EN) | Release 1 Status |
|---|---|---|---|---|
| 1 | `A` | สินทรัพย์ | Assets | Active — 36 groups |
| 2 | `L` | หนี้สิน | Liabilities | Active — 17 groups |
| 3 | `SE` | ส่วนของผู้ถือหุ้น | Shareholders' Equity | Active — 7 groups |
| 4 | `R` | รายได้ | Revenue | Defined — no groups assigned |
| 5 | `E` | ค่าใช้จ่าย | Expense | Defined — no groups assigned |
| 6 | `PL` | กำไรขาดทุน | Profit & Loss | Active — 54 groups (incl. OCI) |

**FS Group Code Format**

Each FS Group code is composed of a letter prefix followed by a numeric suffix, for example `A101`, `L104`, `SE105`, or `PL219B`. The prefix aligns with the FS Group Type code. The numeric suffix has no fixed length. Suffixes may include a trailing letter (e.g., `A104A`, `PL219C`) to distinguish sub-groups within the same major grouping. OCI groups use the `OCI` prefix convention and map to `fs_group_type_id = 6` (PL).

**FS Group Master List**

> ✅ **Confirmed:** The master list below reflects the live seed data loaded into the system as of 2026-02-10. Total: 114 groups across 4 active FS types (A, L, SE, PL). FS Group Types R (Revenue) and E (Expense) are defined in the system but have no groups assigned in Release 1 — all P&L items use type PL (type_id 6).

| FS Code | FS Name (TH) | FS Type ID |
|---|---|---|
| **Assets (A) — type_id 1** | | |
| A101 | เงินสดและรายการเทียบเท่าเงินสด | 1 |
| A102 | เงินลงทุนชั่วคราว | 1 |
| A103 | ลูกหนี้การค้าและลูกหนี้อื่น | 1 |
| A104 | ลูกหนี้เงินให้สินเชื่อ - ส่วนที่ถึงกำหนดชำระภายในหนึ่งปี | 1 |
| A104A | ลูกหนี้เงินให้สินเชื่อ - ส่วนที่ถึงกำหนดชำระภายในหนึ่งปี - ค่าเผื่อฯ | 1 |
| A105 | ลูกหนี้ตามสัญญาเช่าซื้อ - ส่วนที่ถึงกำหนดชำระภายในหนึ่งปี | 1 |
| A105A | ลูกหนี้ตามสัญญาเช่าซื้อ - ส่วนที่ถึงกำหนดชำระภายในหนึ่งปี - ค่าเผื่อฯ | 1 |
| A106 | ลูกหนี้ขายฝาก - ส่วนที่ถึงกำหนดชำระภายในหนึ่งปี | 1 |
| A106A | ลูกหนี้ขายฝาก - ส่วนที่ถึงกำหนดชำระภายในหนึ่งปี - ค่าเผื่อฯ | 1 |
| A107 | ทรัพย์สินรอการขาย - สุทธิ | 1 |
| A108 | เงินให้กู้ยืมระยะสั้น | 1 |
| A109 | เงินให้กู้ยืมระยะยาว-ส่วนที่ถึงกำหนดชำระภายในหนึ่งปี | 1 |
| A110 | สินทรัพย์หมุนเวียนอื่น | 1 |
| A111 | สินค้าคงเหลือ | 1 |
| A112 | ลูกหนี้ประกันผ่อนชำระ | 1 |
| A112A | ลูกหนี้ประกันผ่อนชำระ - ค่าเผื่อฯ | 1 |
| A201 | เงินฝากธนาคารที่มีภาระค้ำประกัน | 1 |
| A202 | เงินลงทุนระยะยาว - เงินฝากประจำ | 1 |
| A203 | เงินลงทุนในบริษัทย่อย | 1 |
| A204 | เงินลงทุนเผื่อขาย | 1 |
| A205 | เงินลงทุนระยะยาวอื่น | 1 |
| A206 | ลูกหนี้เงินให้สินเชื่อ - สุทธิจากส่วนที่ถึงกำหนดชำระภายในหนึ่งปี | 1 |
| A206A | ลูกหนี้เงินให้สินเชื่อ - สุทธิจากส่วนที่ถึงกำหนดชำระภายในหนึ่งปี - ค่าเผื่อฯ | 1 |
| A207 | ลูกหนี้ตามสัญญาเช่าซื้อ - สุทธิจากส่วนที่ถึงกำหนดชำระภายในหนึ่งปี | 1 |
| A207A | ลูกหนี้ตามสัญญาเช่าซื้อ - สุทธิจากส่วนที่ถึงกำหนดชำระภายในหนึ่งปี - ค่าเผื่อฯ | 1 |
| A208 | ลูกหนี้ขายฝาก - สุทธิจากส่วนที่ถึงกำหนดชำระภายในหนึ่งปี | 1 |
| A208A | ลูกหนี้ขายฝาก - สุทธิจากส่วนที่ถึงกำหนดชำระภายในหนึ่งปี - ค่าเผื่อฯ | 1 |
| A209 | เงินให้กู้ยืมระยะยาว-สุทธิจากส่วนที่ถึงกำหนดชำระภายในหนึ่งปี | 1 |
| A210 | อาคารและอุปกรณ์ | 1 |
| A211 | อสังหาริมทรัพย์เพื่อการลงทุน | 1 |
| A212 | สินทรัพย์ไม่มีตัวตน | 1 |
| A212A | สินทรัพย์ไม่มีตัวตน - เพื่อขาย | 1 |
| A213 | สินทรัพย์ภาษีเงินได้รอการตัดบัญชี | 1 |
| A214 | สินทรัพย์ไม่หมุนเวียนอื่น | 1 |
| A215 | สิทธิการใช้สินทรัพย์ | 1 |
| A216 | สินทรัพย์ตราสารอนุพันธ์ | 1 |
| **Liabilities (L) — type_id 2** | | |
| L101 | เงินกู้ยืมระยะสั้นจากสถาบันการเงิน | 2 |
| L102 | เงินกู้ยืมระยะสั้น | 2 |
| L103 | หนี้สินตราสารอนุพันธ์ | 2 |
| L104 | เจ้าหนี้การค้าและเจ้าหนี้อื่น | 2 |
| L105 | ส่วนของเงินกู้ยืมระยะยาวที่ถึงกำหนดชำระภายในหนึ่งปี | 2 |
| L106 | ส่วนของหุ้นกู้ระยะยาวที่ถึงกำหนดชำระภายในหนึ่งปี | 2 |
| L107 | ส่วนของหนี้สินตามสัญญาเช่าซื้อและเช่าการเงินที่ถึงกำหนดชำระภายในหนึ่งปี | 2 |
| L108 | ภาษีเงินได้ค้างจ่าย | 2 |
| L109 | หนี้สินหมุนเวียนอื่น | 2 |
| L109A | ประมาณการค่าชดเชยผลเสียหายจากการค้ำประกัน | 2 |
| L110 | ส่วนของหนี้สินตามสัญญาเช่าที่ถึงกำหนดชำระภายในหนึ่งปี | 2 |
| L201 | เงินกู้ยืมระยะยาว - สุทธิจากส่วนที่ถึงกำหนดชำระภายในหนึ่งปี | 2 |
| L202 | หุ้นกู้ระยะยาว - สุทธิจากส่วนที่ถึงกำหนดชำระภายในหนึ่งปี | 2 |
| L203 | หนี้สินตามสัญญาเช่าซื้อและเช่าการเงิน - สุทธิจากส่วนที่ถึงกำหนดชำระภายในหนึ่งปี | 2 |
| L205 | ประมาณการหนี้สิน - สำรองผลประโยชน์ระยะยาวของพนักงาน | 2 |
| L207 | หนี้สินไม่หมุนเวียนอื่น | 2 |
| L208 | หนี้สินตามสัญญาเช่า - สุทธิจากส่วนที่ถึงกำหนดชำระภายในหนึ่งปี | 2 |
| **Shareholders' Equity (SE) — type_id 3** | | |
| SE101 | ทุนจดทะเบียน ออกจำหน่ายและชำระเต็มมูลค่าแล้ว | 3 |
| SE102 | ส่วนเกินมูลค่าหุ้นสามัญ | 3 |
| SE103 | ใบสำคัญแสดงสิทธิที่จะซื้อหุ้น | 3 |
| SE104 | กำไรสะสมจัดสรรแล้ว - สำรองตามกฎหมาย | 3 |
| SE105 | กำไรสะสมยังไม่ได้จัดสรร | 3 |
| SE106 | องค์ประกอบอื่นของส่วนของผู้ถือหุ้น | 3 |
| SEXXX | None Group | 3 |
| **Profit & Loss (PL) — type_id 6** | | |
| PL101 | รายได้ดอกเบี้ยและค่าธรรมเนียมในการให้บริการสินเชื่อ (Interest and fee income) | 6 |
| PL101A | รายได้ค่าบริการด้านไอที | 6 |
| PL102 | รายได้ค่าธรรมเนียม (Guarantee) | 6 |
| PL102A | รายได้ปรับปรุงจาก STL to EIR (Guarantee) | 6 |
| PL103 | ค่าปรับชำระล่วงหน้า (Prepayment Fee) | 6 |
| PL104A | ค่าปรับล่าช้า (Late Penalty Fee) | 6 |
| PL104B | ค่าติดตามทวงถาม (Collection Fee) | 6 |
| PL105 | รายได้ค่านายหน้า (Commission income) | 6 |
| PL106 | รายได้ค่าธรรมเนียมและค่าบริการอื่น (Other fee & Service income) | 6 |
| PL107 | รายได้อื่น (Other Income) | 6 |
| PL108 | รายได้ค่าบริหารจัดการ (Management fee income) | 6 |
| PL109 | รายได้เงินปันผล (Dividend Income) | 6 |
| PL110 | รายได้จากการขาย (Sales) | 6 |
| PL111 | ต้นทุนขาย (Cost of Good Sold) | 6 |
| PL112 | รายได้ค่าอบรมสัมนา (Training income) | 6 |
| PL113 | ต้นทุนการให้บริการ (Cost of Service) | 6 |
| PL201 | ค่าใช้จ่ายเกี่ยวกับพนักงาน (Personnel expenses) | 6 |
| PL202 | ค่าภาษีอื่น ๆ (Tax expenses) | 6 |
| PL203 | หนี้สูญและหนี้สงสัยจะสูญ (Doubtful expenses) | 6 |
| PL203A | หนี้สูญรับคืน (Recovery from write off) | 6 |
| PL203B | หนี้สูญรับคืน (Recovery from write off) - Insurance | 6 |
| PL204 | ขาดทุนจากการปรับโครงสร้างหนี้ (Loss on debt restructuring) | 6 |
| PL205 | ค่าเสื่อมราคาและค่าตัดจำหน่าย (Depreciation and amortisation expenses) | 6 |
| PL206 | ค่าเดินทาง (Travelling and transportation expenses) | 6 |
| PL207 | ค่าเช่าและค่าบริการ (Rental and service expenses) | 6 |
| PL207A | ค่าบริการอื่น (Other service expenses) | 6 |
| PL208 | ค่าอรรถประโยชน์ (Utilities expenses) | 6 |
| PL209 | ค่าธรรมเนียมวิชาชีพ (Professional fee) | 6 |
| PL210A | ค่าโฆษณาและค่าใช้จ่ายการตลาด (Advertising Online) | 6 |
| PL210B | ค่าโฆษณาและค่าใช้จ่ายการตลาด (Advertising Offline) | 6 |
| PL210C | ค่าโฆษณาและค่าใช้จ่ายการตลาด (Marketing event) | 6 |
| PL211 | ค่าบำรุงรักษา (Maintenance expenses) | 6 |
| PL212 | ค่าประกัน (Insurance expenses) | 6 |
| PL213 | ขาดทุนจากการด้อยค่าทรัพย์สิน (Loss on impairment of assets) | 6 |
| PL214 | ขาดทุนจากการด้อยค่าทรัพย์สินรอจำหน่าย-รถยึด (Loss on impairment of properties foreclosed) | 6 |
| PL217 | ขาดทุนจากการขายทรัพย์สินรอจำหน่าย-รถยึด (Loss on disposals of assets foreclosed) | 6 |
| PL218 | ขาดทุนจากการยึดทรัพย์สินรอการขาย (Loss on Asset Reposess) | 6 |
| PL219 | ค่าใช้จ่ายอื่น (Other expenses) | 6 |
| PL219A | ค่าใช้จ่ายบริการคลาวด์และเทคโนโลยี (Cloud and technology service expenses) | 6 |
| PL219B | ค่าใช้จ่ายอุปกรณ์สำนักงานและวัสดุสิ้นเปลือง (Office equipment & Supply) | 6 |
| PL219C | ค่าธรรมเนียมศาล (Court fee) | 6 |
| PL219D | ค่าใช้จ่ายสำหรับผู้จัดการพื้นที่ (Expenses relating to AM) | 6 |
| PL219E | ขาดทุนจากการขายและตัดจำหน่ายทรัพย์สิน (Loss from disposal and write-off fixed assets) | 6 |
| PL220 | ขาดทุนจากค่าเผื่อการปรับมูลค่าสินค้าและสินค้าล้าสมัย | 6 |
| PL221 | Loss Claim-NTB | 6 |
| PL222 | ค่าใช้จ่ายค่าบริหารจัดการ (Management fee expenses) | 6 |
| PL223 | เงินปันผลจ่าย (Dividend expenses) | 6 |
| PL301 | ต้นทุนทางการเงิน (Financial cost) | 6 |
| PL401 | ค่าใช้จ่ายภาษีเงินได้ (Corporate income tax) | 6 |
| PL401A | ส่วนเปลี่ยนแปลงจากภาษีเงินได้รอตัดฯ (Change in deferred tax) | 6 |
| **Other Comprehensive Income (OCI) — type_id 6** | | |
| OCI101 | Gain(loss) on revaluation/FMV | 6 |
| OCI102 | Income tax relating to OCI | 6 |
| OCI201 | Actuarial Gain/Loss | 6 |
| OCI202 | Income tax relating to Actuarial Gain/Loss | 6 |

#### 4. Acceptance Criteria (BDD)

**Scenario 1: Create a valid FS Group**

Given a user with appropriate API access,
When a POST request is submitted with all required fields (FS Group Type, Group Code, Group Name TH, Group Name EN),
Then the system creates the FS Group record and returns a success response with the new group ID.

**Scenario 2: Create FS Group with missing required field**

Given a user with API access,
When a POST request is submitted with a missing required field (e.g., Group Name EN is absent),
Then the system rejects the request and returns a descriptive error message (e.g., "Group Name - EN is required").

**Scenario 3: Retire an FS Group**

Given an existing active FS Group,
When a deactivation request is submitted via API,
Then the FS Group is marked inactive and can no longer be referenced when creating new GL accounts.

> **Note (Release 1):** This feature is delivered as an API tool only. The accounting team cannot self-manage FS Groups; changes must be submitted as requests to the dev team.

---

### Feature F2: COA Management — General Ledger Create & Update

**Type:** New Feature | **Domain:** Accounting / COA Management
**POB:** POB-2199 | **Priority:** P1

#### 1. Business Context & Value

As an accounting system administrator,
I want to create, update, and retire General Ledger (GL) accounts via API,
So that the chart of accounts remains accurate, non-duplicated, and traceable for all transaction recording.

#### 2. System Context (The Delta Map)

**Current State (As-Is):** GL accounts are managed externally. There is no system-level validation preventing duplicate GL numbers or references to retired FS Groups.

**Target State (To-Be):** The system enforces GL creation rules via API. GL numbers are unique, FS Group references are validated, and the config history between GL and FS Group is recorded.

**Components Impacted:** `master_general_ledger`, `master_fs_group`, `master_sub_ledger_type`, `master_tracking_dimension`

#### 3. Data Specification

**GL Account Number Format**

All GL accounts use a fixed **10-digit numeric** format. The number is structured as four logical segments:

```
X  ·  XXX  ·  XX  ·  XXXX
│     │       │     └── Item (4 digits) — individual account within the sub-group
│     │       └──────── Sub-group (2 digits) — account sub-classification
│     └──────────────── Major group (3 digits) — account group within the type
└────────────────────── Account type (1 digit) — see table below
```

| Leading Digit | Account Type |
|---|---|
| 1 | Asset |
| 2 | Liability |
| 4 | Revenue / Income |
| 5 | Expense |
| 9 | Suspense / Off-balance |

**Examples from the live GL master:**

| Account Number | Name | FS Group | Center Type |
|---|---|---|---|
| 1110000000 | Cash on Hand | A101 | Profit Center |
| 1110100000 | Petty Cash | A101 | Profit Center |
| 1110110000 | Cash Card | A101 | Profit Center |
| 2131000000 | Other Accounts Payable - Related Companies | L104 | Profit Center |
| 2197000000 | Unknown Deposit | L109 | Profit Center |
| 5120000000 | Seizing Expense | PL218 | Cost Center |
| 5210200010 | Advertising - Offline | PL210B | Cost Center |
| 5211217000 | Bank Fee | PL219 | Cost Center |

The account number must be unique across the system. No two GL accounts may share the same 10-digit number regardless of entity or FS Group.

#### 4. Acceptance Criteria (BDD)

**Scenario 1: Create a valid GL account**

Given a user with API access and a valid active FS Group code,
When a POST request is submitted with all required fields (FS Group Code, Account Number, Account Name TH, Account Name EN, Sub-ledger Type),
Then the system creates the GL account and returns a success response.

**Scenario 2: Prevent duplicate GL number**

Given an existing GL account with number "1110100000",
When a new request attempts to create a GL account with the same number,
Then the system rejects the request with an appropriate validation error.

**Scenario 3: Prevent reference to inactive FS Group**

Given an FS Group that has been retired,
When a request attempts to create a GL account under that group,
Then the system rejects the request.

**Scenario 4: Enforce 10-digit account number format**

Given a user with API access,
When a POST request is submitted with an account number that is not exactly 10 digits (e.g., 9 digits or alphanumeric),
Then the system rejects the request with a format validation error.

> **Note (Release 1):** API tool only. The system must record configuration history between GL and FS Group.

---

### Feature F3: COA Management — GL Sub-Ledger Type Assignment

**Type:** New Feature | **Domain:** Accounting / COA Management
**POB:** POB-2194 | **Priority:** P2

#### 1. Business Context & Value

As an accounting system administrator,
I want to assign a sub-ledger type to each GL account that requires counterparty tracking,
So that the system can enforce at transaction entry time whether a Customer or Vendor code must accompany postings to that account.

#### 2. System Context (The Delta Map)

**Current State (As-Is):** There is no system-level flag on GL accounts indicating whether a counterparty code is required. This allows transactions to be posted to reconciliation accounts without a corresponding sub-ledger code, making reconciliation difficult.

**Target State (To-Be):** Each GL account carries a Sub-ledger Type attribute (`Customer`, `Vendor`, or `None`). When a transaction is posted to a GL account with a non-None sub-ledger type, the system requires a counterparty code to be present on that journal line. The counterparty code itself is a fixed value configured at the Accounting Event level (Feature F5) — it is not managed here.

**Components Impacted:** `master_sub_ledger_type`, `master_general_ledger`, `journal_line`, `journal_line_attributes`

#### 3. Data Specification

**Sub-Ledger Type Values**

| Sub-Ledger Type | Meaning | Example GL Account |
|---|---|---|
| `Vendor` | Posting requires a vendor counterparty code | `2131000000` — Other AP - Related Companies |
| `Customer` | Posting requires a customer counterparty code | `1153100000` — Trade AR - Related Companies |
| `None` | No counterparty required | `1110000000` — Cash on Hand |

The counterparty code value itself (e.g., `1020`, `1010`) is not stored in the GL master. It is defined as a fixed value within each Accounting Event configuration and written to `journal_line_attributes` at transaction recording time.

#### 4. Acceptance Criteria (BDD)

**Scenario 1: Assign Sub-Ledger Type during GL creation**

Given a user with API access,
When a GL account is created with Sub-Ledger Type set to `Vendor`,
Then the system records the type and the account is flagged as requiring a vendor code on every posting.

**Scenario 2: Transaction posting to a Vendor GL without a counterparty code**

Given a GL account with Sub-Ledger Type = `Vendor`,
When an accounting transaction is recorded against that account without a vendor counterparty code,
Then the system rejects the journal line with a validation error indicating the counterparty code is required.

**Scenario 3: Transaction posting to a None GL without a counterparty code**

Given a GL account with Sub-Ledger Type = `None`,
When an accounting transaction is recorded against that account without a counterparty code,
Then the system accepts the journal line normally.

---

### Feature F4: COA Management — Cost & Profit Center Create & Update

**Type:** New Feature | **Domain:** Accounting / COA Management
**POB:** POB-2195 | **Priority:** P1

#### 1. Business Context & Value

As an accounting system administrator,
I want to create and maintain cost center and profit center records,
So that accounting transactions can be attributed to the correct organizational units for management reporting.

#### 2. System Context (The Delta Map)

**Current State (As-Is):** Cost and profit center data is not managed within the Bookkeeping system. There is no validation when transactions reference cost/profit centers.

**Target State (To-Be):** The system maintains a structured repository of cost/profit centers. In future releases, this will be auto-synced from LOS (branches) and SAP.

**Components Impacted:** `master_organizational_unit`, `master_unit_type`, `master_business_units`

#### 3. Acceptance Criteria (BDD)

**Scenario 1: Create a cost/profit center unit**

Given a user with API access,
When a request is submitted with Unit Code, Unit Name, Unit Type, and at least one of Cost Center Number or Profit Center Number,
Then the system creates the organizational unit record.

**Scenario 2: Reject duplicate unit code**

Given an existing unit with code "10050121",
When a request submits a new unit with the same code,
Then the system returns an error: "Cost Center Number is duplicated".

**Scenario 3: Disable a unit**

Given an active cost center unit,
When a disable request is submitted via API,
Then the unit is marked inactive and cannot be referenced in new accounting transactions.

---

### Feature F5: Accounting Gateway — Accounting Event Template Config

**Type:** New Feature | **Domain:** Accounting Gateway
**POB:** POB-2192 | **Priority:** P1

#### 1. Business Context & Value

As an accounting system administrator,
I want to configure Accounting Event templates that define the validation rules and double-entry mapping for each business event,
So that all accounting transaction requests are automatically validated and routed to the correct GL accounts without manual intervention.

#### 2. System Context (The Delta Map)

**Current State (As-Is):** Accounting entries are mapped manually. There is no configurable event-to-ledger mapping layer.

**Target State (To-Be):** Each business event type has a configured template specifying the DR/CR GL accounts, optional/required cost/profit centers, sub-ledger rules, and reference field requirements. The gateway validates all incoming requests against this template before creating a journal entry.

**Components Impacted:** `master_accounting_event`, `accounting_config_event`, `accounting_config_field_requirements`, `accounting_event_references`, `master_reference_field`

#### 3. Accounting Event Code Format

All event codes follow the pattern `CCEE NNN` (7 characters, no spaces):

```
[C][EEE][NNN]
 │   │    └── Running number (3 digits, zero-padded, e.g., 001, 002)
 │   └─────── Event category (3 letters)
 └─────────── Company code (1 letter)
```

**Company prefix:**

| Letter | Entity |
|---|---|
| `B` | NTB |
| `I` | NTBI |

**Event category codes:**

| Code | Category |
|---|---|
| `COH` | Cash on Hand |
| `CAJ` | Cash on Hand — Adjustment |
| `PTC` | Petty Cash |
| `PAJ` | Petty Cash — Adjustment |
| `PAY` | Payment |
| `OPE` | Loan Operation |
| `BTC` | Manual / Test (BPTC prefix used for manual UAT entries) |

**Examples:** `BCOH001`, `BPTC014`, `BPAY001`, `BCAJ005`

#### 4. Mapping Logic — How Bookkeeping Uses the Data Sent by the Upstream System

Understanding the mapping logic is critical for the upstream system (Cash Reconciliation / LOS) to send the correct payload for each event. The system derives GL accounts, cost/profit centers, and sub-ledger codes using the following rules.

**Profit/Cost Center Derivation**

The center on each journal line is resolved by one of three mechanisms, and the mechanism is fixed per event side (DR or CR) in the event configuration:

The first mechanism is **branch-derived**: the upstream system must always send the branch code in Reference1 (`รหัสสาขา`). Bookkeeping uses this code to look up the branch's registered profit center or cost center from the organizational unit master. This applies to all lines where the logic column reads "สาขาที่ทำรายการจาก LOS", "สาขาที่ได้รับเงิน", or "สาขาที่เป็นเจ้าของบัตรตามที่ branch register". The distinction between those three Thai phrases only affects which branch's center is used — the transaction branch, the receiving branch, or the card-owning branch — but in all cases the information flows from Ref1.

The second mechanism is a **fixed HQ center** (`1002000000`). Certain event sides always post to HQ's profit center regardless of which branch triggered the transaction. This value is hardcoded in the event configuration; the upstream system does not need to send anything for it.

The third mechanism is a **fixed AM center** (`10050123`). Events involving the Area Manager (AM) cash transfer (`BPTC006`, `BPTC007`, and their ADJ counterparts) post the expense side to a fixed AM cost center. This is also hardcoded in the event configuration.

**Sub-Ledger / Customer / Vendor Code**

Only one GL account across all COH and PTC events carries a vendor sub-ledger requirement: `2131000000` (Other Accounts Payable — Related Companies), confirmed by the GL Master reconciliation flag. This account appears as the CR side of `BCOH018` and `BPAY156`, and as the DR side of their ADJ counterparts (`BCAJ018`).

The sub-ledger codes assigned to that account are fixed in the event configuration:

`1020` — Assigned to `BCOH018` and `BCAJ018`. Represents the insurance-related related-company counterparty (NTB Life entity) on account `2131000000`.

`1010` — Assigned to `BPAY156`. Represents the BNPL-related counterparty on account `2131000000`.

All other accounts in both event catalogues have no reconciliation flag in the GL Master and therefore require no sub-ledger code. The upstream system does not send sub-ledger values; they are hardcoded in the event configuration.

**Reference Fields**

Reference1 (`รหัสสาขา`) is mandatory for every event without exception. It carries the branch code and is the input Bookkeeping needs to derive profit/cost centers. All other reference fields are event-specific and documented in the catalogues below.

**Date Logic**

Posting date and document date are always provided by the upstream system as part of the transaction payload. For most events both dates are the transaction date ("วันที่ทำรายการ"). A small number of events use the bank statement date as the document date ("วันที่ตาม Statement") — specifically `BCOH002`, `BCOH010`, `BCOH011`, `BPTC024`, and `BPTC028`. Events `BCOH017` and `BPTC031` have no date mapping from LOS ("ไม่ map event") and are manual-only adjustments with dates entered manually.

---

#### 5. Event Catalogue — Cash on Hand Branch (NTB)

The tables below use the following notation for the center logic column. **Branch (from Ref1)** means the center is derived by looking up the branch code sent in Reference1. **Fixed: 1002000000** means the HQ profit center is hardcoded and the upstream system sends nothing for that side. Each original event (BCOH/BPAY) is followed immediately by its corresponding adjustment entry (BCAJ) in the same row format, since Bookkeeping treats them identically at the transaction level.

| Event Code | Event Name | Posting Date | Doc Date | DR Acct | DR Name | CR Acct | CR Name | DR Center Type | DR Center Logic | CR Center Type | CR Center Logic | Ref1 | Ref2 | Ref2 Req | Ref3 | Ref3 Req | DR Sub-Ledger | CR Sub-Ledger |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| BCOH001 | Transfer to Cash Card — Loan Cash | Transaction date | Transaction date | 1110110000 | Cash Card | 1110400012 | Current Acc-KBANK-Out-034-3-30540-0 | Profit Center | Branch (from Ref1) — card-owning branch | Profit Center | Fixed: 1002000000 | Branch code | — | — | — | — | — | — |
| BCAJ001 | ADJ — Transfer to Cash Card — Loan Cash | Transaction date | Transaction date | 1110400012 | Current Acc-KBANK-Out-034-3-30540-0 | 1110110000 | Cash Card | Profit Center | Fixed: 1002000000 | Profit Center | Branch (from Ref1) — card-owning branch | Branch code | — | — | — | — | — | — |
| BCOH002 | Withdraw from Cash Card — Loan Cash | Transaction date | Statement date | 1110000000 | Cash on Hand | 1110110000 | Cash Card | Profit Center | Branch (from Ref1) — receiving branch | Profit Center | Branch (from Ref1) — card-owning branch | Branch code | — | — | — | — | — | — |
| BCAJ002 | ADJ — Withdraw from Cash Card — Loan Cash | Transaction date | Transaction date | 1110110000 | Cash Card | 1110000000 | Cash on Hand | Profit Center | Branch (from Ref1) — card-owning branch | Profit Center | Branch (from Ref1) — receiving branch | Branch code | — | — | — | — | — | — |
| BCOH003 | Return to HQ — Loan Cash (KBANK) | Transaction date | Transaction date | 1110401441 | Current-KBANK-Inc Bank-053-1-76377-7 | 1110000000 | Cash on Hand | Profit Center | Fixed: 1002000000 | Profit Center | Branch (from Ref1) | Branch code | — | — | — | — | — | — |
| BCAJ003 | ADJ — Return to HQ — Loan Cash (KBANK) | Transaction date | Transaction date | 1110000000 | Cash on Hand | 1110401441 | Current-KBANK-Inc Bank-053-1-76377-7 | Profit Center | Branch (from Ref1) | Profit Center | Fixed: 1002000000 | Branch code | — | — | — | — | — | — |
| BCOH004 | Cheque Deposit to HQ — Loan Cash | Transaction date | Transaction date | 1110400301 | Saving KBANK Inc 038-1-45556-4 | 1110000000 | Cash on Hand | Profit Center | Fixed: 1002000000 | Profit Center | Branch (from Ref1) | Branch code | — | — | — | — | — | — |
| BCAJ004 | ADJ — Cheque Deposit to HQ — Loan Cash | Transaction date | Transaction date | 1110000000 | Cash on Hand | 1110400301 | Saving KBANK Inc 038-1-45556-4 | Profit Center | Branch (from Ref1) | Profit Center | Fixed: 1002000000 | Branch code | — | — | — | — | — | — |
| BCOH005 | Return to HQ — Loan Cash (Tesco Lotus) | Transaction date | Transaction date | 1156100130 | Other AR - Bill Payment - Lotus | 1110000000 | Cash on Hand | Profit Center | Branch (from Ref1) | Profit Center | Branch (from Ref1) | Branch code | — | — | — | — | — | — |
| BCAJ005 | ADJ — Return to HQ — Loan Cash (Tesco Lotus) | Transaction date | Transaction date | 1110000000 | Cash on Hand | 1156100130 | Other AR - Bill Payment - Lotus | Profit Center | Branch (from Ref1) | Profit Center | Branch (from Ref1) | Branch code | — | — | — | — | — | — |
| BCOH006 | Return to HQ — Loan Cash (KTB) | Transaction date | Transaction date | 1110600011 | Current Acc-KTB-Inc-147-6-01119-2 | 1110000000 | Cash on Hand | Profit Center | Fixed: 1002000000 | Profit Center | Branch (from Ref1) | Branch code | — | — | — | — | — | — |
| BCAJ006 | ADJ — Return to HQ — Loan Cash (KTB) | Transaction date | Transaction date | 1110000000 | Cash on Hand | 1110600011 | Current Acc-KTB-Inc-147-6-01119-2 | Profit Center | Branch (from Ref1) | Profit Center | Fixed: 1002000000 | Branch code | — | — | — | — | — | — |
| BCOH007 | Receive Cash from Market | Transaction date | Transaction date | 1110000000 | Cash on Hand | 4890000000 | Other Income | Profit Center | Branch (from Ref1) | Profit Center | Branch (from Ref1) | Branch code | — | — | — | — | — | — |
| BCAJ007 | ADJ — Receive Cash from Market | Transaction date | Transaction date | 4890000000 | Other Income | 1110000000 | Cash on Hand | Profit Center | Branch (from Ref1) | Profit Center | Branch (from Ref1) | Branch code | — | — | — | — | — | — |
| BCOH008 | Asset Sale Proceeds | Transaction date | Transaction date | 1110000000 | Cash on Hand | 5210600010 | Repair and Maintenance Expense | Profit Center | Branch (from Ref1) | Cost Center | Branch (from Ref1) | Branch code | Asset number | Required | — | — | — | — |
| BCAJ008 | ADJ — Asset Sale Proceeds | Transaction date | Transaction date | 5210600010 | Repair and Maintenance Expense | 1110000000 | Cash on Hand | Cost Center | Branch (from Ref1) | Profit Center | Branch (from Ref1) | Branch code | Asset number | Required | — | — | — | — |
| BCOH009 | Mortgage Fee | Transaction date | Transaction date | 1191900000 | Mortgage Fee Advance | 1110000000 | Cash on Hand | Profit Center | Branch (from Ref1) | Profit Center | Branch (from Ref1) | Branch code | Contract number | Required | — | — | — | — |
| BCAJ009 | ADJ — Mortgage Fee | Transaction date | Transaction date | 1110000000 | Cash on Hand | 1191900000 | Mortgage Fee Advance | Profit Center | Branch (from Ref1) | Profit Center | Branch (from Ref1) | Branch code | Contract number | Required | — | — | — | — |
| BCOH010 | HQ Recall — Loan Cash | Transaction date | Statement date | 1110400011 | Current Acc-KBANK-Inc-034-3-30540-0 | 1110110000 | Cash Card | Profit Center | Fixed: 1002000000 | Profit Center | Branch (from Ref1) — card-owning branch | Branch code | — | — | — | — | — | — |
| BCAJ010 | ADJ — HQ Recall — Loan Cash | Transaction date | Transaction date | 1110110000 | Cash Card | 1110400011 | Current Acc-KBANK-Inc-034-3-30540-0 | Profit Center | Branch (from Ref1) — card-owning branch | Profit Center | Fixed: 1002000000 | Branch code | — | — | — | — | — | — |
| BCOH011 | Bank Fee (Cash Card — Loan Cash) | Transaction date | Statement date | 5211217000 | Bank Fee | 1110110000 | Cash Card | Cost Center | Branch (from Ref1) | Profit Center | Branch (from Ref1) — card-owning branch | Branch code | — | — | — | — | — | — |
| BCAJ011 | ADJ — Bank Fee (Cash Card — Loan Cash) | Transaction date | Transaction date | 1110110000 | Cash Card | 5211217000 | Bank Fee | Profit Center | Branch (from Ref1) — card-owning branch | Cost Center | Branch (from Ref1) | Branch code | — | — | — | — | — | — |
| BCOH012 | Adjustment — Awaiting Customer Refund (Pay Up) | Transaction date | Transaction date | 1110000000 | Cash on Hand | 2198010000 | Other AP - TR | Profit Center | Branch (from Ref1) | Profit Center | Branch (from Ref1) | Branch code | Contract number | Required | — | — | — | — |
| BCAJ012 | ADJ — Awaiting Customer Refund (Pay Up) | Transaction date | Transaction date | 2198010000 | Other AP - TR | 1110000000 | Cash on Hand | Profit Center | Branch (from Ref1) | Profit Center | Branch (from Ref1) | Branch code | Contract number | Required | — | — | — | — |
| BCOH013 | Adjustment — Awaiting Return | Transaction date | Transaction date | 1110000000 | Cash on Hand | 2198000000 | AP - Suspense Account | Profit Center | Branch (from Ref1) | Profit Center | Branch (from Ref1) | Branch code | — | — | — | — | — | — |
| BCAJ013 | ADJ — Awaiting Return | Transaction date | Transaction date | 2198000000 | AP - Suspense Account | 1110000000 | Cash on Hand | Profit Center | Branch (from Ref1) | Profit Center | Branch (from Ref1) | Branch code | — | — | — | — | — | — |
| BCOH014 | Adjustment — Rounding | Transaction date | Transaction date | 1110000000 | Cash on Hand | 4850000000 | Deficit or Surplus | Profit Center | Branch (from Ref1) | Profit Center | Branch (from Ref1) | Branch code | — | — | — | — | — | — |
| BCAJ014 | ADJ — Rounding | Transaction date | Transaction date | 4850000000 | Deficit or Surplus | 1110000000 | Cash on Hand | Profit Center | Branch (from Ref1) | Profit Center | Branch (from Ref1) | Branch code | — | — | — | — | — | — |
| BCOH015 | Adjustment — Unknown Surplus | Transaction date | Transaction date | 1110000000 | Cash on Hand | 2197000000 | Unknown Deposit | Profit Center | Branch (from Ref1) | Profit Center | Branch (from Ref1) | Branch code | — | — | — | — | — | — |
| BCAJ015 | ADJ — Unknown Surplus | Transaction date | Transaction date | 2197000000 | Unknown Deposit | 1110000000 | Cash on Hand | Profit Center | Branch (from Ref1) | Profit Center | Branch (from Ref1) | Branch code | — | — | — | — | — | — |
| BCOH016 | Adjustment — Staff Fraud | Transaction date | Transaction date | 5211208001 | Misc Expense - Loss from Fraud | 1110000000 | Cash on Hand | Cost Center | Branch (from Ref1) | Profit Center | Branch (from Ref1) | Branch code | — | — | — | — | — | — |
| BCAJ016 | ADJ — Staff Fraud | Transaction date | Transaction date | 1110000000 | Cash on Hand | 5211208001 | Misc Expense - Loss from Fraud | Profit Center | Branch (from Ref1) | Cost Center | Branch (from Ref1) | Branch code | — | — | — | — | — | — |
| BCOH017 | Adjustment — Cash on Hand (General) | No date mapping | No date mapping | 5211208000 | Miscellaneous Expense | 1110000000 | Cash on Hand | Cost Center | Branch (from Ref1) | Profit Center | Branch (from Ref1) | Branch code | — | — | — | — | — | — |
| BCAJ017 | ADJ — Cash on Hand (General) | No date mapping | No date mapping | 1110000000 | Cash on Hand | 5211208000 | Miscellaneous Expense | Profit Center | Branch (from Ref1) | Cost Center | Branch (from Ref1) | Branch code | — | — | — | — | — | — |
| BCOH018 | Receive Insurance Payment — Cash | Transaction date | Transaction date | 1110000000 | Cash on Hand | 2131000000 | Other AP - Related Companies | Profit Center | Branch (from Ref1) | Profit Center | Branch (from Ref1) | Branch code | Ref1/Barcode | Optional | Application ID | Optional | — | Fixed: 1020 |
| BCAJ018 | ADJ — Receive Insurance Payment — Cash | Transaction date | Transaction date | 2131000000 | Other AP - Related Companies | 1110000000 | Cash on Hand | Profit Center | Branch (from Ref1) | Profit Center | Branch (from Ref1) | Branch code | Ref1/Barcode | Optional | Application ID | Optional | Fixed: 1020 | — |
| BPAY156 | Receive BNPL Payment — Cash | Transaction date | Transaction date | 1110000000 | Cash on Hand | 2131000000 | Other AP - Related Companies | Profit Center | Branch (from Ref1) | Profit Center | Branch (from Ref1) | Branch code | Contract number | Required | — | — | — | Fixed: 1010 |

> **Note on BCOH018 / BCAJ018 Ref4:** A fourth reference field (policy number) is also documented in the source spec as optional, applicable when the branch collects additional compulsory insurance (พ.ร.บ) payments.

---

#### 6. Event Catalogue — Petty Cash Branch (NTB)

The same center derivation logic applies as described in Section 4. **Branch (from Ref1)** = derived from the branch code in Reference1. **Fixed: 1002000000** = HQ profit center hardcoded. **Fixed: 10050123** = Area Manager cost center hardcoded. Each original event (BPTC) is followed immediately by its corresponding adjustment entry (BPAJ) in the same row format, since Bookkeeping treats them identically at the transaction level.

| Event Code | Event Name | Posting Date | Doc Date | DR Acct | DR Name | CR Acct | CR Name | DR Center Type | DR Center Logic | CR Center Type | CR Center Logic | Ref1 | Ref2 | Ref2 Req |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| BPTC001 | Withdraw from Cash Card — Petty Cash | Transaction date | Transaction date | 1110100000 | Petty Cash | 1110110000 | Cash Card | Profit Center | Branch (from Ref1) — card-owning branch | Profit Center | Branch (from Ref1) — card-owning branch | Branch code | — | — |
| BPAJ001 | ADJ — Withdraw from Cash Card — Petty Cash | Transaction date | Transaction date | 1110110000 | Cash Card | 1110100000 | Petty Cash | Profit Center | Branch (from Ref1) — card-owning branch | Profit Center | Branch (from Ref1) — card-owning branch | Branch code | — | — |
| BPTC002 | Transfer to Cash Card — Petty Cash | Transaction date | Transaction date | 1110110000 | Cash Card | 1110400022 | Current KBANK Out 035-8-87305-7 | Profit Center | Branch (from Ref1) — card-owning branch | Profit Center | Fixed: 1002000000 | Branch code | — | — |
| BPAJ002 | ADJ — Transfer to Cash Card — Petty Cash | Transaction date | Transaction date | 1110400022 | Current KBANK Out 035-8-87305-7 | 1110110000 | Cash Card | Profit Center | Fixed: 1002000000 | Profit Center | Branch (from Ref1) — card-owning branch | Branch code | — | — |
| BPTC003 | Return to HQ — Petty Cash (KTB) | Transaction date | Transaction date | 1110600011 | Current Acc-KTB-Inc-147-6-01119-2 | 1110100000 | Petty Cash | Profit Center | Fixed: 1002000000 | Profit Center | Branch (from Ref1) | Branch code | — | — |
| BPAJ003 | ADJ — Return to HQ — Petty Cash (KTB) | Transaction date | Transaction date | 1110100000 | Petty Cash | 1110600011 | Current Acc-KTB-Inc-147-6-01119-2 | Profit Center | Branch (from Ref1) | Profit Center | Fixed: 1002000000 | Branch code | — | — |
| BPTC004 | Return to HQ — Petty Cash (Tesco Lotus) | Transaction date | Transaction date | 1156100130 | Other AR - Bill Payment - Lotus | 1110100000 | Petty Cash | Profit Center | Branch (from Ref1) | Profit Center | Branch (from Ref1) | Branch code | — | — |
| BPAJ004 | ADJ — Return to HQ — Petty Cash (Tesco Lotus) | Transaction date | Transaction date | 1110100000 | Petty Cash | 1156100130 | Other AR - Bill Payment - Lotus | Profit Center | Branch (from Ref1) | Profit Center | Branch (from Ref1) | Branch code | — | — |
| BPTC005 | Return to HQ — Petty Cash (KBANK) | Transaction date | Transaction date | 1110401441 | Current-KBANK-Inc Bank-053-1-76377-7 | 1110100000 | Petty Cash | Profit Center | Fixed: 1002000000 | Profit Center | Branch (from Ref1) | Branch code | — | — |
| BPAJ005 | ADJ — Return to HQ — Petty Cash (KBANK) | Transaction date | Transaction date | 1110100000 | Petty Cash | 1110401441 | Current-KBANK-Inc Bank-053-1-76377-7 | Profit Center | Branch (from Ref1) | Profit Center | Fixed: 1002000000 | Branch code | — | — |
| BPTC006 | Transfer Petty Cash to AM | Transaction date | Transaction date | 5211313000 | Branch Operation Expenses (AM) | 1110100000 | Petty Cash | Cost Center | Fixed: 10050123 | Profit Center | Branch (from Ref1) | Branch code | Employee ID (AM) | Optional |
| BPAJ006 | ADJ — Transfer Petty Cash to AM | Transaction date | Transaction date | 1110100000 | Petty Cash | 5211313000 | Branch Operation Expenses (AM) | Profit Center | Branch (from Ref1) | Cost Center | Fixed: 10050123 | Branch code | Employee ID (AM) | Optional |
| BPTC007 | Receive Cash from AM | Transaction date | Transaction date | 1110100000 | Petty Cash | 5211313000 | Branch Operation Expenses (AM) | Profit Center | Branch (from Ref1) | Cost Center | Fixed: 10050123 | Branch code | Employee ID (AM) | Optional |
| BPAJ007 | ADJ — Receive Cash from AM | Transaction date | Transaction date | 5211313000 | Branch Operation Expenses (AM) | 1110100000 | Petty Cash | Cost Center | Fixed: 10050123 | Profit Center | Branch (from Ref1) | Branch code | Employee ID (AM) | Optional |
| BPTC008 | Receive Branch Space Rental | Transaction date | Transaction date | 1110100000 | Petty Cash | 1156100120 | Other AR - Space Rental | Profit Center | Branch (from Ref1) | Profit Center | Branch (from Ref1) | Branch code | Lease contract number | Required |
| BPAJ008 | ADJ — Receive Branch Space Rental | Transaction date | Transaction date | 1156100120 | Other AR - Space Rental | 1110100000 | Petty Cash | Profit Center | Branch (from Ref1) | Profit Center | Branch (from Ref1) | Branch code | Lease contract number | Required |
| BPTC009 | Marketing Expense | Transaction date | Transaction date | 5210200010 | Advertising - Offline | 1110100000 | Petty Cash | Cost Center | Branch (from Ref1) | Profit Center | Branch (from Ref1) | Branch code | — | — |
| BPAJ009 | ADJ — Marketing Expense | Transaction date | Transaction date | 1110100000 | Petty Cash | 5210200010 | Advertising - Offline | Profit Center | Branch (from Ref1) | Cost Center | Branch (from Ref1) | Branch code | — | — |
| BPTC010 | Document Delivery Expense | Transaction date | Transaction date | 5210500050 | Mail Expense | 1110100000 | Petty Cash | Cost Center | Branch (from Ref1) | Profit Center | Branch (from Ref1) | Branch code | — | — |
| BPAJ010 | ADJ — Document Delivery Expense | Transaction date | Transaction date | 1110100000 | Petty Cash | 5210500050 | Mail Expense | Profit Center | Branch (from Ref1) | Cost Center | Branch (from Ref1) | Branch code | — | — |
| BPTC011 | Branch Repair & Maintenance | Transaction date | Transaction date | 5210600010 | Repair and Maint Expense | 1110100000 | Petty Cash | Cost Center | Branch (from Ref1) | Profit Center | Branch (from Ref1) | Branch code | — | — |
| BPAJ011 | ADJ — Branch Repair & Maintenance | Transaction date | Transaction date | 1110100000 | Petty Cash | 5210600010 | Repair and Maint Expense | Profit Center | Branch (from Ref1) | Cost Center | Branch (from Ref1) | Branch code | — | — |
| BPTC012 | Stationery & Office Supplies | Transaction date | Transaction date | 5211203000 | Office Supply Expense | 1110100000 | Petty Cash | Cost Center | Branch (from Ref1) | Profit Center | Branch (from Ref1) | Branch code | — | — |
| BPAJ012 | ADJ — Stationery & Office Supplies | Transaction date | Transaction date | 1110100000 | Petty Cash | 5211203000 | Office Supply Expense | Profit Center | Branch (from Ref1) | Cost Center | Branch (from Ref1) | Branch code | — | — |
| BPTC013 | Kitchen & Bathroom Supplies | Transaction date | Transaction date | 5211203000 | Office Supply Expense | 1110100000 | Petty Cash | Cost Center | Branch (from Ref1) | Profit Center | Branch (from Ref1) | Branch code | — | — |
| BPAJ013 | ADJ — Kitchen & Bathroom Supplies | Transaction date | Transaction date | 1110100000 | Petty Cash | 5211203000 | Office Supply Expense | Profit Center | Branch (from Ref1) | Cost Center | Branch (from Ref1) | Branch code | — | — |
| BPTC014 | Water Expense | Transaction date | Transaction date | 5210500020 | Water Expense-Actual | 1110100000 | Petty Cash | Cost Center | Branch (from Ref1) | Profit Center | Branch (from Ref1) | Branch code | — | — |
| BPAJ014 | ADJ — Water Expense | Transaction date | Transaction date | 1110100000 | Petty Cash | 5210500020 | Water Expense-Actual | Profit Center | Branch (from Ref1) | Cost Center | Branch (from Ref1) | Branch code | — | — |
| BPTC015 | Electricity Expense | Transaction date | Transaction date | 5210500010 | Electricity Expense-Actual | 1110100000 | Petty Cash | Cost Center | Branch (from Ref1) | Profit Center | Branch (from Ref1) | Branch code | — | — |
| BPAJ015 | ADJ — Electricity Expense | Transaction date | Transaction date | 1110100000 | Petty Cash | 5210500010 | Electricity Expense-Actual | Profit Center | Branch (from Ref1) | Cost Center | Branch (from Ref1) | Branch code | — | — |
| BPTC016 | Waste Disposal Expense | Transaction date | Transaction date | 5211208000 | Miscellaneous Expense | 1110100000 | Petty Cash | Cost Center | Branch (from Ref1) | Profit Center | Branch (from Ref1) | Branch code | — | — |
| BPAJ016 | ADJ — Waste Disposal Expense | Transaction date | Transaction date | 1110100000 | Petty Cash | 5211208000 | Miscellaneous Expense | Profit Center | Branch (from Ref1) | Cost Center | Branch (from Ref1) | Branch code | — | — |
| BPTC017 | Plants & Garden Supplies | Transaction date | Transaction date | 5211205000 | Office Decoration Expense | 1110100000 | Petty Cash | Cost Center | Branch (from Ref1) | Profit Center | Branch (from Ref1) | Branch code | — | — |
| BPAJ017 | ADJ — Plants & Garden Supplies | Transaction date | Transaction date | 1110100000 | Petty Cash | 5211205000 | Office Decoration Expense | Profit Center | Branch (from Ref1) | Cost Center | Branch (from Ref1) | Branch code | — | — |
| BPTC018 | Vinyl Sign Expense | Transaction date | Transaction date | 5210200010 | Advertising - Offline | 1110100000 | Petty Cash | Cost Center | Branch (from Ref1) | Profit Center | Branch (from Ref1) | Branch code | — | — |
| BPAJ018 | ADJ — Vinyl Sign Expense | Transaction date | Transaction date | 1110100000 | Petty Cash | 5210200010 | Advertising - Offline | Profit Center | Branch (from Ref1) | Cost Center | Branch (from Ref1) | Branch code | — | — |
| BPTC019 | Penalty & Surcharge | Transaction date | Transaction date | 5211100090 | Penalty Fees | 1110100000 | Petty Cash | Cost Center | Branch (from Ref1) | Profit Center | Branch (from Ref1) | Branch code | — | — |
| BPAJ019 | ADJ — Penalty & Surcharge | Transaction date | Transaction date | 1110100000 | Petty Cash | 5211100090 | Penalty Fees | Profit Center | Branch (from Ref1) | Cost Center | Branch (from Ref1) | Branch code | — | — |
| BPTC020 | Copy & Print Expense | Transaction date | Transaction date | 5210200010 | Advertising - Offline | 1110100000 | Petty Cash | Cost Center | Branch (from Ref1) | Profit Center | Branch (from Ref1) | Branch code | — | — |
| BPAJ020 | ADJ — Copy & Print Expense | Transaction date | Transaction date | 1110100000 | Petty Cash | 5210200010 | Advertising - Offline | Profit Center | Branch (from Ref1) | Cost Center | Branch (from Ref1) | Branch code | — | — |
| BPTC021 | Tools & Hardware Supplies | Transaction date | Transaction date | 5211203000 | Office Supply Expense | 1110100000 | Petty Cash | Cost Center | Branch (from Ref1) | Profit Center | Branch (from Ref1) | Branch code | — | — |
| BPAJ021 | ADJ — Tools & Hardware Supplies | Transaction date | Transaction date | 1110100000 | Petty Cash | 5211203000 | Office Supply Expense | Profit Center | Branch (from Ref1) | Cost Center | Branch (from Ref1) | Branch code | — | — |
| BPTC022 | Vehicle Repossession Expense | Transaction date | Transaction date | 5120000000 | Seizing Expense | 1110100000 | Petty Cash | Cost Center | Branch (from Ref1) | Profit Center | Branch (from Ref1) | Branch code | Contract number | Required |
| BPAJ022 | ADJ — Vehicle Repossession Expense | Transaction date | Transaction date | 1110100000 | Petty Cash | 5120000000 | Seizing Expense | Profit Center | Branch (from Ref1) | Cost Center | Branch (from Ref1) | Branch code | Contract number | Required |
| BPTC023 | Drinking Water Expense | Transaction date | Transaction date | 5211208000 | Miscellaneous Expense | 1110100000 | Petty Cash | Cost Center | Branch (from Ref1) | Profit Center | Branch (from Ref1) | Branch code | — | — |
| BPAJ023 | ADJ — Drinking Water Expense | Transaction date | Transaction date | 1110100000 | Petty Cash | 5211208000 | Miscellaneous Expense | Profit Center | Branch (from Ref1) | Cost Center | Branch (from Ref1) | Branch code | — | — |
| BPTC024 | Bank Fee (Cash Card — Petty Cash) | Transaction date | Statement date | 5211217000 | Bank Fee | 1110110000 | Cash Card | Cost Center | Branch (from Ref1) | Profit Center | Branch (from Ref1) — card-owning branch | Branch code | — | — |
| BPAJ024 | ADJ — Bank Fee (Cash Card — Petty Cash) | Transaction date | Transaction date | 1110110000 | Cash Card | 5211217000 | Bank Fee | Profit Center | Branch (from Ref1) — card-owning branch | Cost Center | Branch (from Ref1) | Branch code | — | — |
| BPTC025 | Parade Vehicle Expense | Transaction date | Transaction date | 5210200010 | Advertising - Offline | 1110100000 | Petty Cash | Cost Center | Branch (from Ref1) | Profit Center | Branch (from Ref1) | Branch code | — | — |
| BPAJ025 | ADJ — Parade Vehicle Expense | Transaction date | Transaction date | 1110100000 | Petty Cash | 5210200010 | Advertising - Offline | Profit Center | Branch (from Ref1) | Cost Center | Branch (from Ref1) | Branch code | — | — |
| BPTC026 | Property Tax | Transaction date | Transaction date | 1156131074 | Prepaid Property Tax - Clearing | 1110100000 | Petty Cash | Profit Center | Branch (from Ref1) | Profit Center | Branch (from Ref1) | Branch code | — | — |
| BPAJ026 | ADJ — Property Tax | Transaction date | Transaction date | 1110100000 | Petty Cash | 1156131074 | Prepaid Property Tax - Clearing | Profit Center | Branch (from Ref1) | Profit Center | Branch (from Ref1) | Branch code | — | — |
| BPTC027 | Signboard Tax | Transaction date | Transaction date | 1156131073 | Prepaid Sign Board Tax - Clearing | 1110100000 | Petty Cash | Profit Center | Branch (from Ref1) | Profit Center | Branch (from Ref1) | Branch code | — | — |
| BPAJ027 | ADJ — Signboard Tax | Transaction date | Transaction date | 1110100000 | Petty Cash | 1156131073 | Prepaid Sign Board Tax - Clearing | Profit Center | Branch (from Ref1) | Profit Center | Branch (from Ref1) | Branch code | — | — |
| BPTC028 | HQ Recall — Petty Cash | Transaction date | Statement date | 1110400021 | Current KBANK Inc 035-8-87305-7 | 1110110000 | Cash Card | Profit Center | Fixed: 1002000000 | Profit Center | Branch (from Ref1) — card-owning branch | Branch code | — | — |
| BPAJ028 | ADJ — HQ Recall — Petty Cash | Transaction date | Transaction date | 1110110000 | Cash Card | 1110400021 | Current KBANK Inc 035-8-87305-7 | Profit Center | Branch (from Ref1) — card-owning branch | Profit Center | Fixed: 1002000000 | Branch code | — | — |
| BPTC029 | Adjustment — Unknown Surplus | Transaction date | Transaction date | 1110100000 | Petty Cash | 2197000000 | Unknown Deposit | Profit Center | Branch (from Ref1) | Profit Center | Branch (from Ref1) | Branch code | — | — |
| BPAJ029 | ADJ — Unknown Surplus | Transaction date | Transaction date | 2197000000 | Unknown Deposit | 1110100000 | Petty Cash | Profit Center | Branch (from Ref1) | Profit Center | Branch (from Ref1) | Branch code | — | — |
| BPTC030 | Adjustment — Staff Fraud | Transaction date | Transaction date | 5211208001 | Misc Expense - Loss from Fraud | 1110100000 | Petty Cash | Cost Center | Branch (from Ref1) | Profit Center | Branch (from Ref1) | Branch code | — | — |
| BPAJ030 | ADJ — Staff Fraud | Transaction date | Transaction date | 1110100000 | Petty Cash | 5211208001 | Misc Expense - Loss from Fraud | Profit Center | Branch (from Ref1) | Cost Center | Branch (from Ref1) | Branch code | — | — |
| BPTC031 | Adjustment — Petty Cash (General) | No date mapping | No date mapping | 5211208000 | Miscellaneous Expense | 1110100000 | Petty Cash | Cost Center | Branch (from Ref1) | Profit Center | Branch (from Ref1) | Branch code | — | — |
| BPAJ031 | ADJ — Petty Cash (General) | No date mapping | No date mapping | 1110100000 | Petty Cash | 5211208000 | Miscellaneous Expense | Profit Center | Branch (from Ref1) | Cost Center | Branch (from Ref1) | Branch code | — | — |

#### 6. Acceptance Criteria (BDD)

**Scenario 1: Create a valid accounting event config**

Given an administrator with API access,
When a POST request is submitted with a valid Event Code in the format `CCEE NNN`, Event Name, Company Entity, DR GL, CR GL, and reference field definitions,
Then the system creates the accounting event config and it becomes available for transaction recording.

**Scenario 2: Event code format is enforced**

Given an administrator submitting a new event with a code that does not match the `CCEE NNN` pattern (e.g., a 6-character code or an unrecognised company prefix),
When the request is processed,
Then the system rejects it with a format validation error.

**Scenario 3: Validate reference field requirements**

Given an accounting event configured with Reference1 as mandatory,
When a transaction request for that event is submitted without Reference1,
Then the system rejects the request with a clear validation error.

**Scenario 4: Event with optional sub-ledger**

Given an accounting event where the DR sub-ledger is optional,
When a transaction is submitted without a DR sub-ledger,
Then the system accepts the transaction and records it without the sub-ledger attribute on the DR side.

---

### Feature F6: COA Management — Real-Time Cost & Profit Center Sync from Upstream System

**Type:** New Feature | **Domain:** COA Management / Integration
**POB:** POB-2193 | **Priority:** P1

#### 1. Business Context & Value

As the Bookkeeping system,
I want to create and update cost and profit center records in real time when the upstream system creates or modifies a branch,
So that branch-level accounting attribution is always current and available without manual data entry or batch delays.

#### 2. System Context (The Delta Map)

**Current State (As-Is):** Cost and profit center records for branches must be created manually in the Bookkeeping system after a branch is opened or updated upstream.

**Target State (To-Be):** The upstream system (LOS) calls the Bookkeeping organization unit API in real time whenever a branch is created or its details are updated. The Bookkeeping system creates or updates the corresponding organizational unit record immediately upon receiving the request.

**Components Impacted:** `master_organizational_unit`, `master_unit_type`, `master_business_units`, LOS integration layer

#### 3. API Specification

**Endpoint:** `POST /api/bookkeeping/organization-units`
**Base URL:** `https://bks-api.uat01.ntbx.tech`
**Authentication:** `userId` header (caller's user ID)
**Content-Type:** `application/json`

**Request Headers:**

| Header | Required | Description |
|---|---|---|
| `userId` | Required | ID of the user or system triggering the request. |
| `Content-Type` | Required | Must be `application/json`. |

**Request Body:**

| Field | Required | Type | Description |
|---|---|---|---|
| `unitCode` | Required | String | Unique code identifying the organizational unit (e.g., branch code). |
| `unitName` | Required | String | Display name of the branch or unit. |
| `unitTypeId` | Required | String | Type identifier for the unit (e.g., `"1"` for NTB Branch). |
| `unitCenterDetail` | Required | Array | List of cost/profit center assignments for the unit. |
| `unitCenterDetail[].centerTypeId` | Required | Integer | `1` = Cost Center, `2` = Profit Center. |
| `unitCenterDetail[].centerCode` | Required | String | The cost or profit center code to assign. |

**Example Request:**

```bash
curl --location 'https://bks-api.uat01.ntbx.tech/api/bookkeeping/organization-units' \
--header 'userId: 704' \
--header 'Content-Type: application/json' \
--data '{
    "unitCode": "UATTEST01",
    "unitName": "Branch Test UAT01-1",
    "unitTypeId": "1",
    "unitCenterDetail": [
        {
            "centerTypeId": 1,
            "centerCode": "10051110"
        },
        {
            "centerTypeId": 2,
            "centerCode": "1005111000"
        }
    ]
}'
```

#### 4. Acceptance Criteria (BDD)

**Scenario 1: New branch is created via API**

Given a valid API request with a `unitCode` that does not yet exist in the Bookkeeping system,
When the upstream system calls `POST /api/bookkeeping/organization-units`,
Then the system creates a new organizational unit record with the provided unit code, name, type, and the associated cost and profit center codes, and returns a success response immediately.

**Scenario 2: Existing branch details are updated via API**

Given a valid API request with a `unitCode` that already exists in the Bookkeeping system,
When the upstream system calls `POST /api/bookkeeping/organization-units`,
Then the system updates the existing organizational unit record with the new values provided in the request body and returns a success response.

**Scenario 3: Request with missing required field is rejected**

Given an API request where `unitCode` or `unitName` is absent,
When the request is received,
Then the system returns a validation error response without creating or modifying any record.

**Scenario 4: Request is processed in real time**

Given a branch creation event in the upstream system,
When the upstream system dispatches the API call,
Then the organizational unit record is available in the Bookkeeping system immediately upon successful response, with no batch delay.

---

### Feature F7: Accounting Gateway — Create Single Accounting Transaction (API)

**Type:** New Feature | **Domain:** Accounting Gateway
**POB:** POB-2205 | **Priority:** P1

#### 1. Business Context & Value

As an upstream business system (e.g., AMS, Cash Reconcile),
I want to submit a single accounting transaction request via API,
So that the Bookkeeping system records the validated journal entry in real time.

#### 2. System Context (The Delta Map)

**Current State (As-Is):** Business systems submit accounting data via file-based or manual processes with no real-time validation.

**Target State (To-Be):** A dedicated API endpoint accepts transaction requests, validates them against the Accounting Event config, creates the double-entry journal record, and updates the request status accordingly.

**Components Impacted:** `journal_header`, `journal_line`, `journal_line_attributes`, `journal_header_references`, `master_accounting_event`, `log_request`, `log_response`

#### 3. Acceptance Criteria (BDD)

**Scenario 1: Valid transaction request is processed**

Given an active accounting event config for event code "PTTYCSH_001",
When an API request is submitted with a valid event code, amount, posting date, doc date, and all required reference fields,
Then the system creates a journal header and journal line records (DR and CR), returns a success response, and logs the request.

**Scenario 2: Transaction fails event validation**

Given an event config requiring Reference 1 as mandatory,
When a request is submitted for that event without Reference 1,
Then the system rejects the request, returns a structured error response with an error code, and does not create any journal records.

**Scenario 3: Request log is created regardless of outcome**

Given any inbound API request to the accounting gateway,
When the request is received,
Then a log entry is created in `log_request` before validation begins, regardless of the validation outcome.

---

### Feature F8: Accounting Gateway — Create Transactions by File Upload

**Type:** New Feature | **Domain:** Accounting Gateway
**POB:** POB-2198 | **Priority:** P1

#### 1. Business Context & Value

As an upstream business system (e.g., Cash Reconciliation, automated EOD process),
I want to submit a batch of accounting transactions via a structured file upload,
So that multiple journal entries can be created in a single operation with the GL mapping resolved automatically from the accounting event configuration.

#### 2. System Context (The Delta Map)

**Current State (As-Is):** Bulk accounting data is handled manually or through ad hoc imports with no standardized format or automated validation.

**Target State (To-Be):** The system accepts a structured file, validates the file format as a whole first, then processes each row individually by resolving the full double-entry mapping (GL accounts, cost/profit centers, sub-ledgers) from the accounting event config for the specified event code. Successfully validated rows are recorded to the Accounting Book. A processing summary is sent to the configured notification group on completion.

**Key distinction from F10:** F8 does **not** require the submitter to specify GL accounts, cost/profit centers, or sub-ledgers. These are derived entirely from the accounting event config. F10 (manual create) requires the submitter to specify them explicitly.

**Components Impacted:** `journal_header`, `journal_line`, `journal_line_attributes`, `journal_header_references`, `master_accounting_event`, `accounting_config_event`, `log_request`, `log_response`

#### 3. File Format Specification

Each row represents one accounting transaction request. The system resolves the full DR/CR GL mapping — accounts, cost/profit center, and vendor/customer code — from the accounting event config based on the provided event code. The submitter does not need to know or specify any GL detail.

**Column Definitions (11 columns):**

| # | Column Name | Required | Data Type | Validation Rules |
|---|---|---|---|---|
| 1 | Event code | Required | String | Must match an active Accounting Event registered in the system. The event config governs the full GL mapping for this row. |
| 2 | Posting date | Required | Date | Format `YYYY-MM-DD`. Must not be null. |
| 3 | Doc date | Required | Date | Format `YYYY-MM-DD`. Must not be null. |
| 4 | Amount | Required | Decimal | Must be greater than 0. No comma separator. |
| 5 | Reference1 | Required | String | Must not be null. |
| 6 | Reference2 | Optional | String | — |
| 7 | Reference3 | Optional | String | — |
| 8 | Reference4 | Optional | String | — |
| 9 | Reference5 | Optional | String | — |
| 10 | Reference6 | Optional | String | — |
| 11 | Note | Optional | String | Free-text remark attached to the transaction. |

**File Format Example:**

```
Event code,posting date,Doc date,Amount,Reference1,Reference2,Reference3,Reference4,Reference5,Reference6,Note
BPTC001,2025-01-01,2025-01-01,100,NBI001,,,,,, test1
BPTC002,2025-01-01,2025-01-01,100,NBI001,,,,,, test2
```

#### 4. Processing Logic

The system processes an uploaded file in two sequential stages. Stage 2 is only reached if Stage 1 passes entirely.

**Stage 1 — File Format Validation (file-level gate):** The system checks that all column names are present and correctly named, all required fields are non-null, date fields conform to `YYYY-MM-DD` format, and the amount field is a valid positive number. If any row fails Stage 1, the entire file is rejected and no transactions are created. A rejection notification is sent to the configured notification group.

**Stage 2 — Row-level Event Config Validation:** Each row is validated independently. The system checks that the event code exists and is active in Bookkeeping, then resolves the full double-entry mapping (DR/CR GL accounts, cost/profit center, vendor/customer code, date logic) entirely from the accounting event config. The upstream file does not contain GL accounts, center values, or sub-ledger codes — all of these are derived automatically from config. Rows that pass are recorded to the Accounting Book. Rows that fail are logged with a specific error reason. Processing continues for all remaining rows regardless of individual row failures. A summary notification is sent on completion.

**Validation Error Messages (Stage 2):**

| Condition | Error Message |
|---|---|
| Event code not found in system | `ไม่พบ Event Code บนระบบ` |
| Required field is null | `กรุณาระบุ '[Field]'` |
| Field format is incorrect | `Format '[Field]' ไม่ถูกต้อง` |
| Amount is negative or zero | `amount มีค่าติดลบ` |
| Date format invalid | `Format 'posting date' ไม่ถูกต้อง` |

Multiple validation errors on the same row are concatenated and reported together.

**Notification Format:**

```
Request File name: [filename]
Total request: [N] รายการ
Success request: [N] รายการ
Fail request: [N] รายการ
File Result: [S3 result link]
```

#### 5. Acceptance Criteria (BDD)

**Scenario 1: Valid file is uploaded and all rows are processed successfully**

Given a file that passes all Stage 1 format checks and all rows reference valid, active accounting event codes,
When the file is uploaded to the gateway,
Then the system creates a journal record for each row with the GL mapping resolved from the event config, updates each request status to success, and sends a summary notification to the configured group.

**Scenario 2: File fails Stage 1 format validation — entire file is rejected**

Given a file where at least one row has a missing required column or an incorrectly formatted date,
When the system performs Stage 1 validation,
Then no transactions are created, the entire file is rejected, and a rejection notification is sent to the configured group with a clear error description.

**Scenario 3: File passes Stage 1 but individual rows fail Stage 2 event config validation**

Given a file that passes Stage 1 format validation, but certain rows reference an event code that does not exist in the system,
When the system processes the file row by row in Stage 2,
Then valid rows are recorded to the Accounting Book, invalid rows are logged with specific error messages, and processing continues for all remaining rows.

**Scenario 4: Amount is zero or negative**

Given a file that passes Stage 1 format validation, but a specific row contains an Amount value of zero or less,
When the system validates that row in Stage 2,
Then that row is rejected with the error `amount มีค่าติดลบ` and processing continues for all remaining rows. The file is not halted.

**Scenario 5: Processing summary is sent on completion**

Given any file that has completed processing (fully successful or partially failed),
When Stage 2 processing finishes,
Then the system sends a notification containing the file name, total rows, success count, fail count, and a link to the result file on S3.

> **Note:** This feature will also be used for data migration. Users can prepare historical data in the upload file format for bulk import.

---

### Feature F10: Accounting Gateway — Manual Create Transaction by File Upload (AMS)

**Type:** Enhancement | **Domain:** Accounting Gateway / AMS
**POB:** POB-2206 | **Priority:** P1

#### 1. Business Context & Value

As an accounting user (Preparer) on AMS,
I want to upload a structured file to create multiple manual accounting transactions at once,
So that I can record bulk journal entries efficiently without being bound by standard accounting event configuration.

#### 2. System Context (The Delta Map)

**Current State (As-Is):** Manual accounting adjustments are entered individually or managed outside the system.

**Target State (To-Be):** AMS provides a file upload screen where a Preparer submits a CSV file of manual transactions. The system validates the file format as a whole first, then processes each row individually against master data rules. Successfully validated rows are recorded to the Accounting Book. A processing summary is sent to Google Chat on completion.

**Components Impacted:** `journal_header`, `journal_line`, `journal_line_attributes`, `journal_header_references`, `master_general_ledger`, `master_sub_ledger_type`, `master_organizational_unit`, AMS frontend

#### 3. File Format Specification

The upload file is a delimited flat file. Each row represents one complete double-entry accounting transaction (one DR account and one CR account per row). The `Entity` and `SAP record` columns are excluded from this format.

**Column Definitions (17 columns):**

| # | Column Name | Required | Data Type | Validation Rules |
|---|---|---|---|---|
| 1 | Event code | Required | String | Must match an active Accounting Event registered in the system. |
| 2 | Posting date | Required | Date | Format `YYYY-MM-DD`. Must not be null. |
| 3 | Doc date | Required | Date | Format `YYYY-MM-DD`. Must not be null. |
| 4 | Dr Account Number | Required | String | Must match an active GL account in COA. |
| 5 | Cr Account Number | Required | String | Must match an active GL account in COA. |
| 6 | Dr. Profit center / cost center | Optional | String | If provided, must exist in COA. The system determines whether to map as profit center or cost center based on the DR GL account's master config. |
| 7 | Cr. Profit center / cost center | Optional | String | If provided, must exist in COA. The system determines whether to map as profit center or cost center based on the CR GL account's master config. |
| 8 | Dr. Sub ledger | Optional | String | If provided, must exist in COA and must be valid under the DR GL account. If the DR GL account is of type "No Sub Ledger", this field is ignored. |
| 9 | Cr. Sub ledger | Optional | String | If provided, must exist in COA and must be valid under the CR GL account. If the CR GL account is of type "No Sub Ledger", this field is ignored. |
| 10 | Amount | Required | Decimal | Must be greater than 0. No comma separator. |
| 11 | Reference1 | Required | String | Must not be null. |
| 12 | Reference2 | Optional | String | — |
| 13 | Reference3 | Optional | String | — |
| 14 | Reference4 | Optional | String | — |
| 15 | Reference5 | Optional | String | — |
| 16 | Reference6 | Optional | String | — |
| 17 | Remark | Optional | String | Free-text note attached to the transaction. |

**File Format Example:**

```
Event code,posting date,Doc date,Dr Account Number,Cr Account Number,Dr. Profit center / cost center,Cr. Profit center / cost center,Dr. Sub ledger,Cr. Sub ledger,Amount,Reference1,Reference2,Reference3,Reference4,Reference5,Reference6,Remark
BPTC001,2026-01-27,2026-01-27,1110100000,1110110000,1005000100,1005012300,,150,NBI001,,,,,,,Manual UAT Case No12 D1
BPTC001,2026-01-27,2026-01-27,1110100000,1110110000,1005000100,1005012300,,200,NBI001,,,,,,,Manual UAT Case No12 D1
BPTC006,2026-01-27,2026-01-27,5211313000,1110100000,1005012300,1005012300,,200,NBI001,100,,,,,,Manual UAT Case No12 D1
BPTC008,2026-01-27,2026-01-27,1110100000,1156100120,1005000100,1005012300,,200,NBI001,100,,50009,,,,Manual UAT Case No12 D1
```

#### 4. Processing Logic

**Key distinction from event-driven transaction creation (F8):** In the manual upload path, the user supplies the DR and CR GL account numbers directly. The system does not perform an event config lookup to derive GL accounts, profit/cost centers, or sub-ledger codes. Instead, it consults the GL master for two purposes only: to determine whether each GL account requires a Profit Center or Cost Center (so the center value provided by the user is stored under the correct attribute type), and to determine whether each GL account carries a Customer or Vendor sub-ledger type (so the counterparty code provided by the user is validated accordingly). All other values — amounts, dates, reference fields — are taken directly from the user-supplied file.

The system processes an uploaded file in two sequential stages. Stage 2 is only reached if Stage 1 passes entirely.

**Stage 1 — File Format Validation (file-level gate):** The system checks that all column names are present and correctly named, all required fields are non-null, date fields conform to `YYYY-MM-DD` format, and the amount field is a valid positive number. If any row fails Stage 1, the entire file is rejected and no transactions are created. A rejection notification is sent to the configured Google Chat group.

**Stage 2 — Row-level Master Data Validation:** Each row is validated independently against master data. The system performs the following checks in order, and all applicable errors are collected and concatenated before reporting:

- Event code must exist and be active in Bookkeeping.
- DR Account Number must exist as an active GL account in COA.
- CR Account Number must exist as an active GL account in COA.
- If DR Profit/Cost Center is provided, it must exist in COA. The GL master for the DR account determines whether the value is stored as a Profit Center or Cost Center attribute.
- If CR Profit/Cost Center is provided, it must exist in COA. The GL master for the CR account determines whether the value is stored as a Profit Center or Cost Center attribute.
- If DR Sub-ledger is provided, it must exist in COA. The GL master for the DR account determines the sub-ledger type (Customer / Vendor). If the DR GL account is configured as "No Sub Ledger", the value is silently ignored.
- If CR Sub-ledger is provided, it must exist in COA. The GL master for the CR account determines the sub-ledger type (Customer / Vendor). If the CR GL account is configured as "No Sub Ledger", the value is silently ignored.
- Amount must be greater than 0.
- Date fields must conform to `YYYY-MM-DD` format.

Rows that pass all checks are recorded to the Accounting Book. Rows that fail are logged with specific error messages. Processing continues for all remaining rows regardless of individual failures. A summary notification is sent on completion.

**Validation Error Messages (Stage 2):**

| Condition | Error Message |
|---|---|
| Required field is null | `กรุณาระบุ '[Field]'` |
| Field format is incorrect | `Format '[Field]' ไม่ถูกต้อง` |
| Event code not found in system | `ไม่พบ Event Code บนระบบ` |
| DR Account Number not found in COA | `ไม่พบ Dr. Account บนระบบ` |
| CR Account Number not found in COA | `ไม่พบ Cr. Account บนระบบ` |
| DR cost/profit center not found in COA | `ไม่พบ Dr. Profit center / cost center บนระบบ` |
| CR cost/profit center not found in COA | `ไม่พบ Cr. Profit center / cost center บนระบบ` |
| DR sub-ledger not found in COA | `ไม่พบ Dr. Sub ledger บนระบบ` |
| CR sub-ledger not found in COA | `ไม่พบ Cr. Sub ledger บนระบบ` |
| Amount is negative or zero | `amount มีค่าติดลบ` |
| Amount format is not numeric | `Format amount ไม่ถูกต้อง` |
| Date format invalid | `Format 'posting date' ไม่ถูกต้อง` |

Multiple validation errors on the same row are concatenated and reported together.

**Notification Format (Google Chat):**

```
Request File name: [filename]
Total request: [N] รายการ
Success request: [N] รายการ
Fail request: [N] รายการ
File Result: [S3 result link]
```

#### 5. Acceptance Criteria (BDD)

**Scenario 1: Valid file is uploaded and all rows are processed successfully**

Given a file that passes all Stage 1 format checks and all rows reference valid master data,
When a Preparer uploads the file via the AMS manual upload screen,
Then the system creates a journal record for each row, records the uploader as `created_by`, and sends a Google Chat notification with total, success, and fail counts.

**Scenario 2: File fails Stage 1 format validation — entire file is rejected**

Given a file where at least one row has an incorrect date format (e.g., `14/02/2025` instead of `2026-01-27`) or a missing required column,
When the system performs Stage 1 validation,
Then no transactions are created, the entire file is rejected, and a rejection notification is sent to the configured Google Chat group with a clear error description.

**Scenario 3: File passes Stage 1 but individual rows fail Stage 2 master data checks**

Given a file that passes Stage 1 format validation, but certain rows reference an event code or GL account that does not exist in the system,
When the system processes the file row by row in Stage 2,
Then valid rows are recorded to the Accounting Book, invalid rows are logged with specific error messages, and processing continues for all remaining rows regardless of individual failures.

**Scenario 4: Multiple validation errors on a single row are all reported**

Given a file row where the event code does not exist and the DR cost center does not exist,
When the system validates that row in Stage 2,
Then both errors are reported together for that row: `ไม่พบ Event Code บนระบบ, ไม่พบ Dr. Profit center / cost center บนระบบ`.

**Scenario 5: Amount is zero or negative**

Given a file row where the Amount field contains a negative value or zero,
When the system validates that row in Stage 2,
Then the row is rejected with the error `amount มีค่าติดลบ` and the remaining rows continue to be processed.

**Scenario 6: Sub ledger is provided for a GL account of type "No Sub Ledger"**

Given a file row where the DR GL account is configured as "No Sub Ledger" and a DR Sub Ledger value is provided,
When the system processes that row,
Then the sub ledger value is silently ignored and the transaction is created without a sub ledger attribute on the DR side.

---

### Feature F12: Accounting Book — Transaction Journal Record (Raw Data)

**Type:** New Feature | **Domain:** Accounting Book
**POB:** POB-2201 | **Priority:** P1

#### 1. Business Context & Value

As the Bookkeeping system,
I want a high-integrity raw data store for every accounting transaction,
So that there is a complete, immutable audit trail of all double-entry records including sub-ledger and cost/profit center attributes.

#### 2. System Context (The Delta Map)

**Current State (As-Is):** No centralized, validated raw journal store exists within the Bookkeeping system.

**Target State (To-Be):** Every successfully validated accounting transaction is persisted to the journal store with full DR/CR GL detail, optional sub-ledger and cost/profit center attributes, reference data, posting date, doc date, and an active status flag.

**Components Impacted:** `journal_header`, `journal_line`, `journal_line_attributes`, `journal_header_references`

#### 3. Acceptance Criteria (BDD)

**Scenario 1: Transaction is stored with full double-entry detail**

Given a successfully validated accounting transaction request,
When the transaction is created,
Then the system persists a `journal_header` record linked to the accounting event, and two `journal_line` records (one DR, one CR) each with the correct GL account, accounting type, and amount.

**Scenario 2: Reference data is stored**

Given a transaction request containing values for Reference 1 and Reference 2,
When the transaction is created,
Then `journal_header_references` records are created for each reference field with the corresponding values.

**Scenario 3: Active status defaults to true**

Given a newly created accounting transaction,
When the record is stored,
Then `is_active` (or equivalent active status flag) defaults to `true` and can be updated to `false` by an authorized user.

---

### Feature F13: Accounting Book — Book of Record (Summary / Pivot View)

**Type:** New Feature | **Domain:** Accounting Book / Reporting
**POB:** POB-2203 | **Priority:** P2

#### 1. Business Context & Value

As an accounting team member,
I want to query a structured summary view of all accounting transactions and their SAP posting statuses,
So that I can monitor the state of the general ledger and produce the data needed for accounting work without needing direct access to raw journal tables.

#### 2. System Context (The Delta Map)

**Current State (As-Is):** There is no aggregated or formatted view of accounting data available for the accounting team.

**Target State (To-Be):** The system maintains a `pivot_view` table (RDS) that provides a formatted, queryable summary of accounting transactions including company code, event code, posting date, DR/CR account details, reference fields, SAP grouping document, and status. The view is updated when transactions are created or when JV transactions are processed.

**Components Impacted:** `pivot_view`, `pivot_view_refresh_log`, `journal_header`, `sap_transaction`

#### 3. Acceptance Criteria (BDD)

**Scenario 1: New transaction appears in pivot view**

Given a new accounting transaction that has been successfully created on the accounting book,
When the `pivot_view` is queried,
Then the transaction appears with correct company code, event code, account numbers, amount, and reference values.

**Scenario 2: JV transaction status is reflected in pivot view**

Given an accounting transaction that has been grouped into a JV transaction and posted to SAP,
When the posting result is updated in the system,
Then the corresponding `pivot_view` record reflects the updated SAP status and group document reference.

**Scenario 3: Refresh log is maintained**

Given any refresh operation on the `pivot_view`,
When the refresh completes (successfully or with error),
Then a record is written to `pivot_view_refresh_log` with the refresh type, status, records processed, and timestamp.

> **Note (Release 1):** Delivered as a database table for querying and export. A UI display with a manual refresh button is planned for a future release.

---

### Feature F14: SAP Connector — Accounting Transaction Outbound

**Type:** New Feature | **Domain:** SAP Connector
**POB:** POB-2191 | **Priority:** P1

#### 1. Business Context & Value

As the SAP integration process,
I want every accounting transaction created in the Bookkeeping system to have a tracked outbound status,
So that the system knows which transactions are pending SAP posting and can group them efficiently into JV transactions.

#### 2. System Context (The Delta Map)

**Current State (As-Is):** There is no formal tracking of which journal entries have been submitted to SAP or what their posting outcome was.

**Target State (To-Be):** When an accounting transaction is created on the Accounting Book, a corresponding `sap_transaction` record is created with initial grouping status `Ungroup` (or `Do Not Post` for adjustment/migration transactions). This status is updated as the transaction progresses through JV grouping and SAP posting.

**Components Impacted:** `sap_transaction`, `sap_transaction_history`, `master_inherited_posting_status`, `master_group_lifecycle_status`, `journal_header`

#### 3. Acceptance Criteria (BDD)

**Scenario 1: Normal transaction creates outbound record with Ungroup status**

Given a standard accounting transaction created via API or file upload,
When the journal record is written to the Accounting Book,
Then a `sap_transaction` record is created with grouping lifecycle status = `Ungroup` and posting status = `Ungroup`.

**Scenario 2: Adjustment transaction creates outbound record with Do Not Post status**

Given an accounting transaction flagged as type `adjustment` or `migration` in the request,
When the journal record is written,
Then the `sap_transaction` record is created with grouping lifecycle status = `Do Not Post`.

**Scenario 3: Status history is maintained**

Given a `sap_transaction` record that changes status over time (e.g., Ungroup → Grouped → Posted),
When each status change occurs,
Then a new record is appended to `sap_transaction_history` capturing the previous state and timestamp.

---

### Feature F15: SAP Connector — Journal Voucher Export (Auto Batch)

**Type:** New Feature | **Domain:** SAP Connector
**POB:** POB-2202 | **Priority:** P2

#### 1. Business Context & Value

As the SAP posting process,
I want the Bookkeeping system to automatically consolidate unposted accounting transactions into JV files at T+1,
So that SAP FI receives a structured, validated JV upload file each business day without manual intervention.

#### 2. System Context (The Delta Map)

**Current State (As-Is):** JV files for SAP are created manually and are not generated from a structured accounting transaction store.

**Target State (To-Be):** A nightly batch job consolidates all `sap_transaction` records with grouping status `Ungroup` and active status `true` into JV Transactions, grouped by a defined set of attributes. Each JV group is assigned a reference group identifier. A JV Upload file conforming to the 43-column SAP format is generated (max 900 group documents per file) and placed on S3.

**Components Impacted:** `sap_jv_transaction`, `sap_jv_transaction_detail`, `sap_config_jv_transaction`, `sap_config_jv_transaction_file`, `sap_file`, `sap_transaction`

#### 3. JV Upload File Specification

**File Naming:** `NTBxxx_yyyymmdd` — e.g., `NTB001_20250814`
**S3 Path (Auto Batch):** `daily-report/Accounting/JVupload/Bookkeeping/auto/yyyy/mm/NTBxxx_yyyymmdd`
**Max Group Documents per File:** 900. When the total exceeds 900, a new file is created and the group document running number restarts from 1.

Each grouped JV transaction produces exactly two rows in the file (one DR line, one CR line). All amount values within a group must net to zero.

**Column Definitions (43 columns total):**

| # | Column Name | Field Length | Required | Rules |
|---|---|---|---|---|
| 1 | Grp Doc | 3 | Yes | Sequential group document number per file. |
| 2 | Company Code | 4 | Yes | Exactly 4 digits: NTB=`1000`, NTBPL=`1010`, NTBI=`1020`, NTBX=`1030`. |
| 3 | Ledger | 2 | No | Left blank for standard entries. |
| 4 | Posting Date | 8 | Yes | Format `ddmmyyyy`, no separator, exactly 8 digits. |
| 5 | Document Date | 8 | Yes | Format `ddmmyyyy`, no separator, exactly 8 digits. |
| 6 | Doc Type | 2 | Yes | `SA` for HQ entries; `PT` for branch entries. |
| 7 | Currency | 3 | Yes | Always `THB`. |
| 8 | Exchange Rate | (4,5) | No | Left blank for THB transactions. |
| 9 | Reference | 16 | Yes | Populated with the Accounting Event Code (e.g., `BOPE001`). |
| 10 | Doc Header Text | 25 | Yes | Format: `[YYYYMMDD][EventCode]_[RunningNo6digits]`. e.g., `20250828BOPE001_000000001`. Primary key for SAP result file matching. |
| 11 | Business Place/Value Date | 4 | Yes | Always `0`. |
| 12 | BP Branch Code (BCODE) | 5 | No | Branch code of the business place. |
| 13 | BP Tax ID (TAXID) | 13 | No | Tax identification number. |
| 14 | Posting Key | 2 | Yes | See Posting Key Rules table below. |
| 15 | Account | 7 | Yes | GL account number. If the event involves a Customer or Vendor, use the Customer/Vendor number instead of the GL number. |
| 16 | Special G/L Indicator | 1 | No | — |
| 17 | Amount | (13,2) | Yes | Positive for DR posting keys (40, 01, 21). Negative for CR posting keys (50, 31, 11). No comma separator. All lines within a group must sum to zero. |
| 18 | Tax Code | 2 | No | — |
| 19 | Tax Base Amount | (13,2) | No | — |
| 20 | Payment Term | 4 | No | — |
| 21 | Baseline Date/Value Date | 8 | No | Format `ddmmyyyy`, no separator. |
| 22 | Payment Method | 1 | No | — |
| 23 | Cost Center | 10 | No | Maps from DR/CR cost center on the accounting transaction. |
| 24 | Profit Center | 10 | No | Maps from DR/CR profit center on the accounting transaction. |
| 25 | Order | 12 | No | — |
| 26 | Assignment | 18 | No | — |
| 27 | Text | 50 | No | Transaction description / remark. |
| 28–31 | WHT Type/Code/Base/Amount 1 | Various | No | Withholding tax set 1. |
| 32–35 | WHT Type/Code/Base/Amount 2 | Various | No | Withholding tax set 2. |
| 36–43 | Name1, Name2, Street, City, Postal Code, Country, Bank Key, Bank Account | Various | No | Vendor/Customer address and banking fields. |

**Posting Key Rules:**

| Account Type | DR Key | CR Key |
|---|---|---|
| Customer | 01 | 11 |
| Vendor | 21 | 31 |
| General Ledger | 40 | 50 |

**Grouping Attributes:** Accounting transactions are consolidated into one group document when all of the following match: Company Code, Posting Date, Document Date, Doc Type, Currency, Reference (Event Code), Business Place, Posting Key, Account, Baseline Date, Profit Center, and Text.

**Data Flow Example — raw transactions → JV file output:**

```
Source Book transactions (Cases 6, 7, 8 — Event BPAY001, Company 1000):
  Case 6:  DR 1110000000  +100,000  |  CR 9111000021  -100,000
  Case 7:  DR 1110000000   +55,000  |  CR 9111000021   -55,000
  Case 8:  DR 1110000000  +350,530  |  CR 9111000021  -350,530

JV File output (Group Doc 4 = 2 rows, amount nets to zero):
  Grp Doc 4 | Co 1000 | 27082025 | SA | THB | BPAY001 | 20250828BPAY001_000000001
    Row 1:  PK 40  Account 1110000000  Amount +505,530.00  (DR line)
    Row 2:  PK 50  Account 9111000021  Amount -505,530.00  (CR line)
```

#### 4. Acceptance Criteria (BDD)

**Scenario 1: Batch groups eligible transactions and assigns reference numbers**

Given accounting transactions with grouping status `Ungroup`, active status `true`, and within the target date range,
When the nightly batch job runs,
Then the system groups transactions by the defined attributes, creates `sap_jv_transaction` records with Doc Header Text in the format `YYYYMMDDEventCode_000000NNN`, and updates each included transaction's grouping status to `Grouped`.

**Scenario 2: JV file is generated and placed on S3**

Given a set of JV transactions produced by the batch job,
When file generation runs,
Then the system produces a flat file conforming to the 43-column format above, with a maximum of 900 group documents per file, named `NTBxxx_yyyymmdd`, and stores it at `daily-report/Accounting/JVupload/Bookkeeping/auto/yyyy/mm/`.

**Scenario 3: File splits correctly when group count exceeds 900**

Given a batch producing 1,050 group documents,
When the file is generated,
Then two files are created: the first contains groups 1–900 and the second contains groups 1–150, with the running number restarting at 1, each with a distinct file name stored on S3.

**Scenario 4: All amount values within each group net to zero**

Given any generated JV file,
When the file content is validated before S3 upload,
Then the sum of all amount values within each group document equals zero; any group that does not balance is flagged and excluded from the file.

**Scenario 5: Already-grouped transactions are excluded**

Given a mix of transactions where some have grouping status `Grouped` and others `Ungroup`,
When the batch job runs,
Then only the `Ungroup` transactions are included in the new JV file.

---

### Feature F16: SAP Integration — Upload SAP Transfer Result

**Type:** New Feature | **Domain:** SAP Connector / AMS
**POB:** POB-2624 | **Priority:** P2

#### 1. Business Context & Value

As an accounting user on AMS,
I want to upload the SAP posting result file after JV submission,
So that the Bookkeeping system reflects the accurate Success/Fail status for each JV transaction group and the accounting team can act on failures promptly.

#### 2. System Context (The Delta Map)

**Current State (As-Is):** SAP posting results are tracked manually and do not feed back into the accounting system.

**Target State (To-Be):** AMS provides an upload screen for the SAP result file (CSV). The system validates the file format as a whole first, then processes each row individually — matching `Doc Header Text` values to JV transaction groups, updating their posting statuses, and sending a processing summary to Google Chat. Status overwrites are supported, allowing a previously successful group to be updated to failed and vice versa.

**Components Impacted:** `sap_jv_transaction`, `sap_jv_transaction_detail`, `sap_file`, AMS frontend

#### 3. File Format Specification

**Supported Format:** CSV
**Required Columns:** 5 (Row, Doc Header Text, Status, Key, Error Message)

| # | Column Name | Required | Validation Rules |
|---|---|---|---|
| 1 | Row | Required | Must be a positive integer. Must not be null. |
| 2 | Doc Header Text | Required | Format `YYYYMMDDEventCode_000000NNN`. Must not be null. Must exist as a JV transaction group in the Bookkeeping system. |
| 3 | Status | Required | Must be exactly `Success` or `Fail`. No other values are accepted. |
| 4 | Key | Required | Must be `Debit` or `Credit`. |
| 5 | Error Message | Optional | Free-text error detail from SAP. May contain multiple error codes (e.g., `error code 1: xxx, error code 2: yyy`). |

**File Format Example:**

```
Row,Doc Header Text,Status,Key,Error Message
1,20250828BOPE001_000000001,Fail,Debit,Invalid GL
2,20250828BOPE001_000000001,Fail,Credit,Invalid GL
3,20250828BOPE001_000000002,Success,Debit,
4,20250828BOPE001_000000002,Success,Credit,
5,20250828BPAY021_000000001,Success,Debit,
6,20250828BPAY021_000000001,Success,Credit,
7,20250828BPAY001_000000001,Fail,Credit,"error code 1: fafasffafs, error code 2: fasfafasfa"
```

**Status Resolution Rule:** A JV transaction group is marked `Fail` if any row belonging to that group carries `Status = Fail`. The group is marked `Success` only when all rows are `Success`. Each group must have both a Debit and a Credit row; a group missing either side is flagged with `Key ไม่ครบ`.

#### 4. Processing Logic

Processing follows the same two-stage pattern as other file upload features.

**Stage 1 — File Format Validation (file-level gate):** The system checks that all five required column headers are present, all required fields are non-null, the `Status` field contains only `Success` or `Fail`, and `Doc Header Text` conforms to the expected format. If any row fails Stage 1, the entire file is rejected and no statuses are updated.

**Stage 2 — Row-level Data Validation:** Each row is processed independently. The system verifies that the `Doc Header Text` exists in the Bookkeeping system. Rows that pass are used to update the corresponding JV transaction group status. Rows that fail are logged with a specific error message. Processing continues for all remaining rows regardless of individual row failures.

**Validation Error Messages:**

| Condition | Error Message |
|---|---|
| Required field is null | `กรุณาระบุ [Field]` |
| Doc Header Text not found in system | `ไม่พบ Doc Header Text บนระบบ` |
| Doc Header Text format is invalid | `Format Doc Header Text ไม่ถูกต้อง` |
| Status value is not Success or Fail | `Format Status ไม่ถูกต้อง` |
| Group is missing Debit or Credit row | `Key ไม่ครบ` |

Multiple validation issues on the same row are concatenated and reported together (e.g., `ไม่พบ Doc Header Text บนระบบ, กรุณาระบุ Status`).

**Notification Format (Google Chat):**

On successful processing:
```
Date: ddmmyyyy hh:mm:ss
Uploaded file name: [filename]
Message: รายการทั้งหมด [N] รายการ - สำเร็จ [N] รายการ, ไม่สำเร็จ [N] รายการ
Link: [S3 result link]
```

When rows fail data validation, a link to the result file on S3 is included so users can review the error detail.

#### 5. Acceptance Criteria (BDD)

**Scenario 1: Valid file updates all JV transaction group statuses correctly**

Given a valid result file where all rows pass format and data validation,
When the file is uploaded via AMS,
Then the system updates each matched JV transaction group to `Success` or `Fail` per the status resolution rule, records the error message for each `Fail` row, and sends a Google Chat notification with the file name, total rows, success count, and fail count.

**Scenario 2: Any row with Fail status marks the entire group as Fail**

Given a JV transaction group `20250828BOPE001_000000001` with two rows where both the Debit and Credit rows carry `Status = Fail`,
When the file is processed,
Then the group's posting status is updated to `Fail` and the error messages from both rows are recorded.

**Scenario 3: Status overwrite is supported in both directions**

Given a JV transaction group previously marked `Success`,
When a new result file is uploaded marking that same group as `Fail`,
Then the system overwrites the status to `Fail` and records the new error message. The reverse — overwriting `Fail` to `Success` — is equally supported.

**Scenario 4: Doc Header Text not found in system**

Given a result file row whose `Doc Header Text` does not correspond to any JV transaction group in Bookkeeping,
When the system processes that row in Stage 2,
Then the row is flagged with `ไม่พบ Doc Header Text บนระบบ` and the remaining rows continue to be processed.

**Scenario 5: Invalid Status value causes row-level failure**

Given a result file row where `Status = Pending` instead of `Success` or `Fail`,
When the system validates that row,
Then the row is rejected with `Format Status ไม่ถูกต้อง` and the remaining rows continue to be processed.

**Scenario 6: File format validation failure rejects the entire file**

Given a file uploaded with missing required column headers,
When the system performs Stage 1 validation,
Then the entire file is rejected, no statuses are updated, and the user receives the error `Column ไม่ครบถ้วน, กรุณาตรวจสอบ Format`.

**Scenario 7: Group missing Debit or Credit row is flagged**

Given a JV transaction group that has only a Debit row in the result file but no Credit row,
When the system processes the file,
Then the group is flagged with `Key ไม่ครบ` and its status is not updated.

---

### Feature F17: SAP Connector — Manual Create JV File (AMS)

**Type:** New Feature | **Domain:** SAP Connector / AMS
**POB:** POB-2363 | **Priority:** TBC

#### 1. Business Context & Value

As an accounting user (Preparer) on AMS,
I want to manually trigger JV transaction creation for a selected company, date range, and set of event codes,
So that I can generate a JV Upload file on demand without waiting for the nightly batch, for cases requiring immediate or ad hoc SAP posting.

#### 2. System Context (The Delta Map)

**Current State (As-Is):** JV file creation is only possible via the automated nightly batch job.

**Target State (To-Be):** AMS provides a "Create JV" screen where a user configures the company code, posting date range, and event codes to include. The system consolidates eligible accounting transactions, creates JV transactions, generates one or more JV files (max 900 group documents per file), stores them on S3 under the Manual JV folder, and sends a Google Chat notification with the result summary and file name list.

**Components Impacted:** `sap_jv_transaction`, `sap_config_jv_transaction`, `sap_file`, `sap_transaction`, AMS frontend

#### 3. AMS Screen & User Flow

The screen provides the following filter inputs. The user must select at least one filter criterion before triggering the generation.

**Step 1 — Select company:** Dropdown selection of Company Code (`1000` NTB / `1010` NTBPL / `1020` NTBI / `1030` NTBX).

**Step 2 — Select posting date range:** Posting Date From and Posting Date To date pickers.

**Step 3 — Select event codes:** Multi-select list of event codes available under the selected company code.

**Step 4 — Review summary:** The system displays a summary table of the eligible transactions matching the filter, grouped by Company Code, Event Code, Posting Date, and Doc Date with summed amounts. The user reviews the total before confirming.

**Step 5 — Confirm and generate:** The user clicks confirm. A pop-up confirmation dialog appears before the system proceeds. On confirmation, the system groups the transactions, creates JV transactions and JV files, and sends a notification.

**Grouping logic:** Transactions are grouped by Company Code, Event Code, Posting Date, and Doc Date — consistent with the auto batch grouping in F15.

#### 4. File Naming Convention

Each generated JV file follows this naming pattern:

```
cccc_yyyymmdd_hhmmss_manual
```

Where `cccc` is the company code, `yyyymmdd` is the date the file was generated, and `hhmmss` is the time of generation. Each file produced in the same request receives a distinct timestamp suffix, incrementing by one second per file.

**Example — one request producing three files:**

```
1000_20251024_100143_manual
1000_20251024_100144_manual
1000_20251024_100145_manual
```

**S3 Path (Manual):** The file is placed in the Manual JV folder on S3, separate from the auto batch path. The user can download the file directly from the AMS JV display screen.

#### 5. Notification Format (Google Chat)

**When generation succeeds:**

```
Date: ddmmyyyy hh:mm:ss
File name:
1000_20251024_100143_manual.txt
1000_20251024_100144_manual.txt
1000_20251024_100145_manual.txt
Message:
Amount total: [N]
Success: [N]
Fail: [N]
```

**When no eligible transactions are found:**

```
Date: ddmmyyyy hh:mm:ss
File name: null
Message:
Amount total: 0
Success: 0
Fail: 0
```

#### 6. Acceptance Criteria (BDD)

**Scenario 1: User triggers manual JV creation with valid filter**

Given eligible accounting transactions with grouping status `Ungroup` matching the selected company code, date range, and event codes,
When the user confirms the Create JV request from AMS,
Then the system consolidates those transactions into JV transactions using the grouping logic, generates one or more JV files (max 900 group documents per file) with names in the format `cccc_yyyymmdd_hhmmss_manual`, uploads them to the Manual S3 folder, updates the grouping status of all included transactions to `Grouped`, and sends a Google Chat notification listing all generated file names and the result summary.

**Scenario 2: Multiple files are generated when group count exceeds 900**

Given a manual JV request that produces more than 900 group documents,
When the system generates the files,
Then multiple files are created with distinct sequential timestamp suffixes (e.g., `100143`, `100144`, `100145`), each containing at most 900 group documents, and all file names are listed in the Google Chat notification.

**Scenario 3: No eligible transactions found for the selected filter**

Given a filter combination that returns no accounting transactions with grouping status `Ungroup`,
When the user confirms the request,
Then no JV transaction or file is created, and a Google Chat notification is sent with `File name: null` and all counts as zero.

**Scenario 4: Generated files are downloadable from AMS**

Given one or more JV files successfully created via a manual trigger,
When the user navigates to the JV display screen in AMS,
Then all generated files are listed with download links available.

**Scenario 5: Accounting transactions are marked Grouped after file generation**

Given accounting transactions included in a manual JV run,
When the JV file generation completes successfully,
Then all included transactions have their grouping status updated to `Grouped` and are excluded from future auto batch and manual JV runs.

---

### Feature F18: SAP Connector — Cancel JV Transaction

**Type:** New Feature | **Domain:** SAP Connector
**POB:** POB-2389 | **Priority:** TBC

#### 1. Business Context & Value

As a system support user,
I want to cancel a JV transaction batch that has failed to post to SAP,
So that the underlying accounting transactions are released back to `Ungroup` status and can be re-grouped into a new, corrected JV batch.

#### 2. System Context (The Delta Map)

**Current State (As-Is):** There is no mechanism to cancel a JV transaction batch once it has been created. When a batch fails to post to SAP, its accounting transactions remain locked and cannot be re-processed.

**Target State (To-Be):** An authorized user can cancel a JV transaction batch whose Posting Status is `FAILED`. The cancellation updates the JV Voucher Status from `CONFIRMED` to `CANCELLED`, releases all linked accounting transactions back to Grouping Status `Ungroup`, and preserves the `FAILED` Posting Status as a historical record of the last SAP communication attempt.

**Components Impacted:** `sap_jv_transaction`, `sap_transaction`, `master_jv_transaction_voucher_status`, `master_jv_transaction_posting_status`, `master_group_lifecycle_status`

#### 3. Status Reference

**JV Transaction — Voucher Status** (internal lifecycle of the batch itself):

| Status | Definition | Triggering Event |
|---|---|---|
| `CONFIRMED` | Initial state. The batch is created and valid. Linked accounting transactions are locked. | System creates a new JV batch via grouping process. |
| `CANCELLED` | Final void state. The batch has been cancelled by a user. Linked accounting transactions are released for re-grouping. | Authorized user cancels a batch whose Posting Status is `FAILED`. |

**JV Transaction — Posting Status** (result of SAP communication):

| Status | Definition | Triggering Event |
|---|---|---|
| `PENDING` | Default state. The batch has been created and is awaiting SAP upload. | System creates a new JV batch. |
| `SUCCESS` | SAP confirmed the batch was successfully posted. Terminal state. | System receives success confirmation from SAP. |
| `FAILED` | SAP rejected the batch. User intervention required. | System receives an error response from SAP. |

**Combined Workflow:**

When a JV batch is created: Voucher Status = `CONFIRMED`, Posting Status = `PENDING`. After successful SAP upload: Posting Status → `SUCCESS` (Voucher Status unchanged — permanently locked). After failed SAP upload: Posting Status → `FAILED`. User then cancels: Voucher Status → `CANCELLED`, Posting Status remains `FAILED` as historical record, and linked accounting transactions revert to Grouping Status `Ungroup`.

#### 4. Acceptance Criteria (BDD)

**Scenario 1: Cancel a failed JV batch releases accounting transactions**

Given a JV transaction batch with Voucher Status = `CONFIRMED` and Posting Status = `FAILED`,
When an authorized user triggers the cancel action,
Then the JV Voucher Status is updated to `CANCELLED`, the Posting Status remains `FAILED`, and all linked accounting transactions have their Grouping Lifecycle Status reverted to `Ungroup`.

**Scenario 2: Cancel is rejected when Posting Status is not FAILED**

Given a JV transaction batch with Posting Status = `PENDING` or `SUCCESS`,
When an authorized user attempts to cancel the batch,
Then the system rejects the request with a validation error — cancel is only permitted when the Posting Status is `FAILED`.

**Scenario 3: Released transactions become available for re-grouping**

Given a JV batch that has been cancelled and its transactions reverted to `Ungroup`,
When the next JV grouping run executes,
Then those transactions are eligible to be included in a new JV batch.

> **Note (Release 1):** Delivered as a backdoor API. A self-service cancel flow via AMS is planned for a future release.

---

### Feature F19: Cancel Accounting Transaction

**Type:** New Feature | **Domain:** Accounting Book
**POB:** POB-2417 | **Priority:** TBC

#### 1. Business Context & Value

As a system support user,
I want to cancel an individual accounting transaction that is erroneous,
So that it is permanently excluded from JV grouping and SAP posting without requiring cancellation of an entire JV batch.

#### 2. System Context (The Delta Map)

**Current State (As-Is):** No mechanism exists to mark individual accounting transactions as invalid within the system. An erroneous transaction can only be excluded by cancelling the entire JV batch it belongs to.

**Target State (To-Be):** An authorized user can cancel an individual accounting transaction. Cancellation sets the Data Integrity Status to `Void` and the Grouping Lifecycle Status to `Do Not Post`, permanently excluding the transaction from all future JV grouping runs and SAP posting. The Inherited Posting Status is also updated to `Do Not Post` to prevent the transaction from appearing in the SAP connector pipeline.

**Components Impacted:** `journal_header`, `sap_transaction`, `sap_transaction_history`, `master_group_lifecycle_status`, `master_inherited_posting_status`

#### 3. Status Reference

**Accounting Transaction — Inherited Posting Status** (SAP connector view, recorded on `sap_transaction`):

| Status | Alternative Name | Definition | Triggering Event |
|---|---|---|---|
| `Ungroup` | Unbatched | Not yet part of a JV batch. No SAP posting status. | Default when transaction is created and Grouping Status is Unbatched. |
| `Submitted` | In Progress | Parent JV batch has been sent to SAP, awaiting response. | Parent JV batch Posting Status changes to `PENDING` → `SUBMITTED`. |
| `Posted` | Success | Parent JV batch was successfully posted to SAP. | Parent JV batch Posting Status changes to `SUCCESS`. |
| `Failed` | Error | Parent JV batch failed to post to SAP. | Parent JV batch Posting Status changes to `FAILED`. |
| `Do Not Post` | — | Transaction must not be posted to SAP and will never be grouped. | 1) Created as a transaction type that requires no SAP posting. 2) Cancellation of the accounting transaction. |

**Accounting Transaction — Grouping Lifecycle Status** (recorded on `sap_transaction`):

| Status | Alternative Name | Definition | Triggering Event |
|---|---|---|---|
| `Ungroup` | Available | Initial state. Transaction is available for JV batching. | 1) New active transaction is created. 2) Parent JV batch is cancelled — transaction is released. |
| `Grouped` | Consolidated | Transaction is locked in a JV batch. Cannot be selected for another batch. | System includes this transaction in a new JV batch. |
| `Do Not Post` | Excluded | Final state. Transaction is permanently excluded from JV batching. Exists for internal records only. | 1) Transaction created as a type that does not require SAP posting. 2) Cancel accounting transaction action. |

**Accounting Transaction — Data Integrity Status** (recorded on `journal_header`):

| Status | Alternative Name | Definition | Triggering Event |
|---|---|---|---|
| `Active` | Valid | Default state. Transaction is valid and eligible for all standard processing. | New accounting transaction is created. |
| `Void` | Cancelled | Transaction has been manually flagged as erroneous. Permanently excluded from JV batching and reporting. | Authorized user cancels the transaction. |

#### 4. Acceptance Criteria (BDD)

**Scenario 1: Cancel an ungrouped accounting transaction**

Given an accounting transaction with Data Integrity Status = `Active` and Grouping Lifecycle Status = `Ungroup`,
When an authorized user cancels the transaction,
Then the Data Integrity Status is updated to `Void`, the Grouping Lifecycle Status is updated to `Do Not Post`, and the Inherited Posting Status is updated to `Do Not Post`.

**Scenario 2: Cancelled transaction is excluded from JV grouping**

Given an accounting transaction with Grouping Lifecycle Status = `Do Not Post`,
When the next JV grouping run executes,
Then the transaction is not included in any new JV batch.

**Scenario 3: Cancel is rejected for a transaction already in a JV batch**

Given an accounting transaction with Grouping Lifecycle Status = `Grouped` (locked in a JV batch),
When an authorized user attempts to cancel the transaction,
Then the system rejects the request with a validation error — the parent JV batch must be cancelled first (Feature F18) before the individual transaction can be cancelled.

**Scenario 4: Cancelled transaction does not appear in the pivot view as postable**

Given an accounting transaction that has been cancelled and has Inherited Posting Status = `Do Not Post`,
When the pivot view is queried,
Then the transaction's status is reflected as `Do Not Post` and it is not included in any pending SAP upload queue.

---

## 7. Open Items & Flags

The following items require clarification or confirmation before the corresponding features can be fully implemented.

| # | Feature | Open Item | Owner |
|---|---|---|---|
| ~~1~~ | ~~F1~~ | ~~FS Group master list confirmed from ACC.~~ ✅ Closed — live seed data loaded 2026-02-10. Full catalogue (115 groups) documented in F1 data specification. | ~~BA / ACC~~ |
| 2 | General | API specification document has not yet been provided. Feature acceptance criteria for API request/response contracts are incomplete and will be updated when the spec is received. | Dev |

---

*This document will be updated as the API specification and additional technical details are provided. Sections marked with `> Flag` or listed in Section 7 require input before implementation can proceed.*