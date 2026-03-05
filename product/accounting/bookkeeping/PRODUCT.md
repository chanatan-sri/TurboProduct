# Product: Bookkeeping

**Product Name**: Bookkeeping — Internal Accounting System
**Project Code**: 1020
**Status**: 📝 Draft
**Executive Owner**: CFO
**Product Owners**: Phasathon & Pojchara
**Portfolio**: Accounting → [PORTFOLIO](../PORTFOLIO.md)
**Last Updated**: 2026-03-04

---

## Problem Statement

NTB business units (AMS, LOS, Cash Reconcile) generate accounting transactions that must be recorded in SAP FI. Currently, this process is fragmented and manual — there is no centralised validation layer, no structured Chart of Accounts enforcement, and no automated JV file generation. This creates risks of incorrect GL mapping, duplicate entries, and compliance exposure on financial statements.

---

## Value Proposition

Bookkeeping is the single source of truth for all double-entry ledger records across NTB. It provides:

- **For Accounting Team:** A structured, validated journal store with a queryable Book of Record. Eliminates manual spreadsheet reconciliation.
- **For Finance / CFO:** Accurate, auditable financial data flowing directly to SAP FI with full reference trails.
- **For Upstream Systems (AMS, LOS, Cash Reconcile):** A standardised API and file upload gateway for submitting accounting events without needing to know GL account details.

---

## Product Boundary

**This product IS responsible for:**
- Receiving and validating accounting transaction requests (via API and file upload)
- Enforcing Chart of Accounts rules (FS Groups, GL accounts, Cost/Profit Centers, Sub-ledger types)
- Recording double-entry journal entries in the Accounting Book (raw journal store)
- Maintaining a pivot/summary view (Book of Record) for querying and reporting
- Grouping validated transactions into JV batches and generating JV upload files for SAP FI
- Processing SAP FI posting result files and updating transaction statuses
- Managing Accounting Event configuration (GL mapping rules per event code)
- Providing Cost & Profit Center master data (synced from upstream)

**This product IS NOT responsible for:**
- SAP FI itself — Bookkeeping only produces files for SAP; SAP FI owns its own processing
- Upstream business logic in AMS, LOS, or Cash Reconcile — those systems own their own transaction triggers
- Customer-facing financial reporting or statements — that belongs to SAP FI / Finance reporting tools
- Payroll, procurement, or ERP functions outside of accounting transaction recording
- Loan account lifecycle management — owned by Core Banking (Platform Portfolio)

**This product RECEIVES from:**
- AMS (Accounting Management System) → manual transaction entry or file upload → via API / file
- LOS (Loan Origination System) → automated accounting event files → via S3 file
- Cash Reconcile → automated accounting event files → via S3 file
- SAP FI → posting result files → via AMS manual upload

**This product SENDS to:**
- SAP FI → JV upload files (max 900 document groups per file) → via S3

---

## Capability Registry

| # | Capability | Owner | Description | Status |
|---|-----------|-------|-------------|--------|
| 1 | [COA Management](capabilities/coa-management/CAPABILITY.md) | Phasathon & Pojchara | Manage Chart of Accounts: FS Groups, GL accounts, Sub-ledger types, Cost & Profit Centers | 📝 Draft |
| 2 | [Accounting Gateway](capabilities/accounting-gateway/CAPABILITY.md) | Phasathon & Pojchara | Inbound transaction intake: API single entry, upstream file upload, manual AMS file upload, event config management | 📝 Draft |
| 3 | [Accounting Book](capabilities/accounting-book/CAPABILITY.md) | Phasathon & Pojchara | Raw journal store and Book of Record (pivot view): store, query, and cancel accounting transactions | 📝 Draft |
| 4 | [SAP Connector](capabilities/sap-connector/CAPABILITY.md) | Phasathon & Pojchara | Outbound SAP FI integration: JV batch grouping, JV file generation, result processing, JV cancellation | 📝 Draft |
| 5 | [Master Data](capabilities/master-data/CAPABILITY.md) | Phasathon & Pojchara | Organisational reference data: entities, accounting types, reference field definitions, unit/branch structures | 📝 Draft |

---

## Product Metrics & KPIs

| Metric | Target | Notes |
|--------|--------|-------|
| Transaction validation pass rate | > 99% | % of submitted transactions that pass Stage 1 + Stage 2 validation |
| SAP JV posting success rate | > 99.5% | % of JV batches posted to SAP FI without error |
| JV file generation latency | Same business day | Time from transaction creation to JV file on S3 |
| Pivot view staleness | < 5 minutes | Max lag between raw journal write and pivot view refresh |
| COA update SLA (dev team) | < 1 business day | Time from COA change request to deployment (Release 1 API-only constraint) |

---

## High-Level User Flow

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

---

## Release Plan

| Release | Scope | Status |
|---------|-------|--------|
| Release 1 | Core Foundation: COA management API, Accounting Gateway, Accounting Book, SAP Connector (F1–F19) | 📝 In Progress |
| Release 2 | Parallel Run & Reconciliation: run alongside current system for accuracy validation | ⏳ Planned |
| Release 3 | Full Cutover: decommission legacy accounting process | 🔲 TBC |

---

## Integration Map

```
Bookkeeping (Project 1020)

  RECEIVES:
  ← AMS            : POST /transactions (single), file upload (bulk manual) → Accounting Gateway
  ← LOS            : S3 file (event code + amount + refs, no GL) → Accounting Gateway
  ← Cash Reconcile : S3 file (event code + amount + refs, no GL) → Accounting Gateway
  ← SAP FI         : SAP result file (JV posting status) → SAP Connector via AMS upload

  SENDS:
  → SAP FI         : JV upload file (.csv / fixed format, max 900 doc groups) → via S3
```
