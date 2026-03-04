# CHANGELOG 001 — Core Banking: Initial Product Definition

**Date**: 2026-03-04
**Layer Affected**: Product, Portfolio
**Author**: Product Owner session

---

## What Changed

### New: `product/platform/core-banking/PRODUCT.md`
Initial product definition created for Core Banking.

**Key decisions recorded:**
- Core Banking is a **TurboProduct-owned system**, not an external vendor system. A PRODUCT.md is warranted.
- Core Banking belongs in the **Platform Portfolio** (not Credit) because it is foundational infrastructure serving all credit products — not a customer-facing domain product. Analogy: DaVinci is to customer identity what Core Banking is to financial account state.
- Core Banking is defined as **one product with four capabilities**, not split into multiple products. The four capabilities (Loan Account Management, Payment Hierarchy, Interest/Fee Calculation, DPD Engine) all share the same loan account context. Splitting across products creates consistency risk, particularly over DPD authority.

**Product boundary set:**
- IS: account lifecycle, balance store, payment hierarchy, interest/fee accrual, DPD engine
- IS NOT: origination (Onigiri), customer identity (DaVinci), collection workflow (Collections), repayment servicing (Loan Servicing TBD), branch tasks (Sensei)

### New: `product/platform/core-banking/capabilities/` — 4 capability stubs
- `loan-account-management/CAPABILITY.md` — account lifecycle, balance store, transaction ledger
- `payment-hierarchy-engine/CAPABILITY.md` — configurable payment allocation rules
- `interest-fee-calculation/CAPABILITY.md` — accrual, penalty interest, fee amortization
- `dpd-engine/CAPABILITY.md` — authoritative DPD counter, LoanDPDChanged events

### Updated: `product/platform/PORTFOLIO.md`
- Investment thesis expanded to include Core Banking alongside DaVinci as the two foundational Platform products.
- Constituent products table updated.
- Cross-product dependencies updated: Core Banking event flow to DaVinci documented.
- Strategic roadmap updated with Core Banking now/next/later items.
- Key risks updated to include Core Banking as a blocking dependency for Collections and Loan Servicing.

### Updated: `product/PORTFOLIO_CATALOG.md`
- Core Banking registered as a Platform Portfolio product (status: Draft, codename: TBD).
- Directory structure updated.

---

## Rationale

**Why this was needed**: Core Banking was previously referenced only as an external integration boundary in other products (Onigiri, DaVinci). Confirming that TurboProduct owns and builds this system requires it to be formally registered as a product with a defined boundary, capability registry, and integration map. Without this definition, downstream products (Collections, Loan Servicing) could not be specced against a clear Core Banking contract.

**Why Platform Portfolio**: The Platform Portfolio is the home for shared infrastructure owned by engineering/data teams with no direct revenue attribution. Core Banking fits this profile: it serves Credit, and potentially future portfolios. Its investment is governed differently from domain products (Credit, Operations). Placing it in Credit would conflate it with the origination domain it serves.

---

## Decision Log

| Decision | Rationale | Alternatives Rejected |
|----------|-----------|----------------------|
| One product, four capabilities (not split) | Context coherence: DPD, balance, payment allocation all depend on the same account record | Split into multiple products → consistency risk, DPD authority ambiguity |
| Platform Portfolio (not Credit) | Foundational infrastructure pattern, multi-product consumer, engineering-owned | Credit Portfolio → would conflate infrastructure with domain |
| Codename TBD | No codename assigned in this session | Japanese-themed codename to be assigned at next Portfolio review |

---

## Open Items Carried Forward

| Item | Owner | Priority |
|------|-------|----------|
| Assign codename | Platform PO | Medium |
| Create ARCHITECTURE.md: API contracts, data model, event schema registry | Core Banking PO | High (blocking Collections + Loan Servicing integration) |
| Resolve: payment hierarchy config owned by Core Banking or Onigiri Campaign Config? | Core Banking PO + Credit PO | Medium |
| Resolve: write-off approval flow ownership | Core Banking PO + Collections PO | Medium |
| Feature decomposition under all four capabilities | Core Banking Engineering | Later |
