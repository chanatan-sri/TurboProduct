# CHANGELOG 001 — Initial Product Definition

**Date**: 2026-03-04
**Layer**: Portfolio + Product + Capability
**Author**: Phasathon & Pojchara
**Status**: Final

---

## What Changed

### Portfolio Layer
- Created **Accounting** portfolio (`product/accounting/PORTFOLIO.md`)
- Accounting is a new portfolio covering internal financial operations and accounting automation
- Business Owner: CFO

### Product Layer
- Created **Bookkeeping** product (`product/accounting/bookkeeping/PRODUCT.md`)
- Project Code: 1020
- Defined product boundary: single source of truth for double-entry ledger records across NTB
- Defined integration map: receives from AMS, LOS, Cash Reconcile → sends to SAP FI
- Created `ATLAS.md` as consolidated living source of truth

### Capability Layer
Created 5 capability stubs corresponding to the 5 functional domains from `bookkeeping_overview.md`:

| Capability | Path |
|-----------|------|
| COA Management | `capabilities/coa-management/CAPABILITY.md` |
| Accounting Gateway | `capabilities/accounting-gateway/CAPABILITY.md` |
| Accounting Book | `capabilities/accounting-book/CAPABILITY.md` |
| SAP Connector | `capabilities/sap-connector/CAPABILITY.md` |
| Master Data | `capabilities/master-data/CAPABILITY.md` |

### Portfolio Catalog
- Added Accounting portfolio and Bookkeeping product to `product/PORTFOLIO_CATALOG.md`

---

## Rationale

The Bookkeeping system (Project 1020) was onboarded into the TurboProduct documentation system to align with the Atomic Architecture standard used across all NTB products. Source document: `bookkeeping_overview.md` (Release 1 Product Requirements Overview, last updated 2026-03-01).

---

## Next Actions

| # | Action | Layer | Owner |
|---|--------|-------|-------|
| 1 | Assign F-numbers to Master Data features | Capability | Phasathon & Pojchara |
| 2 | Resolve open questions in each CAPABILITY.md | Capability | Phasathon & Pojchara |
| 3 | Create FEATURE_<name>.md files for each P1 feature (F1–F8, F10, F12–F16) | Feature | Engineering Owner |
| 4 | Create ARCHITECTURE.md for Bookkeeping (API contracts, data models, SAP JV file schema) | Product | Engineering Owner |
| 5 | Clarify DaVinci dependency for master_entity (Platform portfolio cross-dependency) | Product | Platform PO |
