# Portfolio Catalog

> Master registry of all portfolios and their constituent products.
> This is the single entry point for any cross-product or cross-portfolio work.
> **Read this first before any product-level session.**
>
> Last Updated: 2026-03-04 — Added Accounting portfolio (Bookkeeping)

---

## Portfolio Registry

| Portfolio | Product | Codename | Status | Portfolio Def | Summary |
|-----------|---------|----------|--------|---------------|---------|
| **Credit** | Loan Origination System | Onigiri (おにぎり) | 📝 Draft | [PORTFOLIO](credit/PORTFOLIO.md) | All-in-one loan origination platform. Smart Form intake → fixed-topology underwriting workflow → campaign-driven risk assessment → disbursement. Integrates with Matcha, Wasabi, DaVinci, Sensei, Core Banking, NCB. |
| **Operations** | Document Verification Service | Matcha (抹茶) | ✅ Active | [PORTFOLIO](operations/PORTFOLIO.md) | Universal domain-agnostic document verification engine. 4-state task lifecycle, flexible rule configuration, SHA-256 change detection, async car check integration, re-flow deduplication, AI-first routing via Wasabi. |
| **Operations** | AI Document Verification | Wasabi (わさび) | 📝 Draft | [PORTFOLIO](operations/PORTFOLIO.md) | Stateless LLM-based document verification. 4-stage pipeline: quality → type classification → instruction verification → report assembly. Operates outside Matcha; returns results to Onigiri for early warnings and Matcha for routing. |
| **Operations** | Branch Operations Orchestration | Sensei (先生) | 📝 Draft | [PORTFOLIO](operations/PORTFOLIO.md) | Centralized branch worklist and task orchestration. Aggregates work from Sensei playbooks, external service requests (Onigiri, Matcha), and supervisor tasks into a single prioritized queue. Contact compliance via DaVinci events. |
| **Platform** | Customer & Product Master Data | DaVinci (ダヴィンチ) | 📝 Draft | [PORTFOLIO](platform/PORTFOLIO.md) | Enterprise Golden Record. Consent-based visibility (PDPA), event-driven sync, customer data change management, collection contact compliance, data consolidation engine (field-level authority, no-data-loss), data resolution workflow (3-tier). |
| **Platform** | Core Banking | TBD | 📝 Draft | [PORTFOLIO](platform/PORTFOLIO.md) | Authoritative financial ledger for all loan accounts. Loan account lifecycle management, configurable payment hierarchy engine, interest & fee calculation, DPD engine. Single source of truth for account balance and delinquency status across all credit products. |
| **Accounting** | Bookkeeping | Bookkeeping | 📝 Draft | [PORTFOLIO](accounting/PORTFOLIO.md) | Internal accounting platform. Single source of truth for double-entry ledger records across NTB. COA management, accounting gateway (API + file upload), raw journal store, Book of Record (pivot view), and SAP FI integration via JV file export. Integration layer between AMS/LOS/Cash Reconcile and SAP FI. |

---

## Directory Structure

```
product/
├── PORTFOLIO_CATALOG.md          ← You are here
│
├── credit/
│   ├── PORTFOLIO.md
│   └── onigiri/
│       ├── PRODUCT.md
│       ├── ATLAS.md              ← Detailed source of truth (migration period)
│       └── capabilities/
│
├── operations/
│   ├── PORTFOLIO.md
│   ├── matcha/
│   │   ├── PRODUCT.md
│   │   ├── ATLAS.md
│   │   ├── ARCHITECTURE.md
│   │   └── capabilities/
│   ├── wasabi/
│   │   ├── PRODUCT.md
│   │   ├── ATLAS.md
│   │   └── capabilities/
│   └── sensei/
│       ├── PRODUCT.md
│       ├── ATLAS.md
│       └── capabilities/
│
├── platform/
│   ├── PORTFOLIO.md
│   ├── davinci/
│   │   ├── PRODUCT.md
│   │   ├── ATLAS.md
│   │   ├── Architecture.md
│   │   └── capabilities/
│   └── core-banking/
│       ├── PRODUCT.md
│       └── capabilities/
│
└── accounting/
    ├── PORTFOLIO.md
    └── bookkeeping/
        ├── PRODUCT.md
        ├── ATLAS.md
        └── capabilities/
```

---

## Migration Status

> Products are being migrated from flat ATLAS.md structure to the 4-layer Atomic Architecture.
> During migration, `ATLAS.md` in each product folder remains the detailed source of truth.
> `PRODUCT.md` and `CAPABILITY.md` files are extractions. `FEATURE_<name>.md` files are pending engineering owner assignment.

| Layer | Status |
|-------|--------|
| Portfolio (PORTFOLIO_CATALOG + PORTFOLIO.md) | ✅ Complete |
| Product (PRODUCT.md) | ✅ Complete |
| Capability (CAPABILITY.md) | ✅ Complete (stubs) |
| Feature (FEATURE_*.md) | ⏳ Pending — requires engineering owner assignment |
