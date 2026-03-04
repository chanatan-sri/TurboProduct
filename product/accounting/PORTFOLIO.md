# Portfolio: Accounting

**Strategic Domain**: Internal Financial Operations & Accounting Automation
**Business Owner**: CFO
**Status**: Active Investment
**Last Updated**: 2026-03-04

---

## Investment Thesis

The Accounting portfolio holds the internal financial infrastructure that centralises, validates, and automates the recording of accounting transactions across all NTB business units.

Its primary product, **Bookkeeping**, acts as the integration layer between upstream business systems (AMS, LOS, Cash Reconcile) and the downstream financial reporting system (SAP FI). It is the single source of truth for all double-entry ledger records within NTB.

**Why a separate portfolio?** Accounting products have fundamentally different governance and ownership patterns from operational or credit products:
- Owned and governed by the Finance/Accounting team (not product teams).
- No direct customer-facing value proposition — value is measured by accuracy, auditability, and compliance.
- Regulatory exposure is high: incorrect journal entries directly impact financial statements and external audits.
- SAP FI is an external dependency that imposes strict file format and batch constraints not present in other portfolios.

Conflating this portfolio with Platform or Operations would obscure the Finance team's accountability and the distinct compliance obligations of financial data.

---

## Constituent Products

| Product | Codename | Status | PRODUCT.md | Description |
|---------|----------|--------|------------|-------------|
| Bookkeeping | Bookkeeping | 📝 Draft | [PRODUCT](bookkeeping/PRODUCT.md) | Internal accounting platform. Centralises double-entry ledger recording, validates transactions against COA config, and integrates with SAP FI via JV file export. Single source of truth for all accounting transactions across NTB. |

---

## Portfolio-Level OKRs

| Objective | Key Result | Target |
|-----------|-----------|--------|
| Establish single source of truth for accounting transactions | % of accounting transactions passing through Bookkeeping vs. manual processes | 100% post-cutover |
| Ensure accuracy of journal records | % of transactions posted to SAP without error | > 99.5% |
| Eliminate manual JV file preparation | Time from transaction creation to JV file generation | Automated — same business day |
| Enable financial auditability | % of transactions with full reference trail (event code, doc date, posting date, GL accounts) | 100% |

---

## Cross-Product Dependencies

```
Accounting Portfolio ← Upstream Systems
  Bookkeeping ← AMS (Accounting Management System) : Manual transactions, file upload → via API / file
  Bookkeeping ← LOS (Loan Origination System)       : Automated accounting events → via API / S3 file
  Bookkeeping ← Cash Reconcile                       : Automated accounting events → via API / S3 file

Accounting Portfolio → Downstream Systems
  Bookkeeping → SAP FI : JV upload file (max 900 document groups) → via S3 / manual upload
  Bookkeeping → SAP FI : Receives SAP posting result file → via AMS file upload
```

---

## Strategic Roadmap

| Horizon | Initiative | Owner |
|---------|-----------|-------|
| **Now** | Release 1 — Core Foundation: COA management, Accounting Gateway, Accounting Book, SAP Connector | Phasathon & Pojchara |
| **Next** | Release 2 — Parallel Run & Reconciliation: Run alongside current system to validate accuracy | Phasathon & Pojchara |
| **Later** | Release 3 — Full Cutover: Decommission legacy accounting process | TBC |

---

## Key Risks and Constraints

| Risk | Severity | Mitigation |
|------|----------|-----------|
| SAP FI is a hard external dependency | Critical | JV file format must exactly match SAP FI expectations. Any schema change requires SAP team coordination. |
| Incorrect GL mapping in event config causes wrong journal entries | Critical | Event config validation must be enforced at configuration time, not at posting time. |
| COA is managed via API only (no self-service UI) in Release 1 | High | All COA changes go through the dev team. Must have clear SLA for COA update requests. |
| Pivot view refresh is asynchronous | Medium | Book of Record may lag behind raw journal records. Consumers must be aware of eventual consistency. |
| SAP result file upload is manual in Release 1 | Medium | Accounting team dependency — delayed uploads will cause transactions to remain in PENDING status. |
