# ITEM: Insurance Section in Smart Form

**Status:** Concept
**Capability:** Insurance Integration
**Product:** Onigiri
**Owner:** TBD
**Created:** 2026-03-30
**Last Updated:** 2026-03-30

---

## What & Why
New section in the Smart Form's Loan Setup stage that provides the UI container for insurance selection. Contains two conditionally visible sub-sections: Credit Insurance (plan picker) and Voluntary/Compulsory Insurance (reference number input with multi-reference support). Includes a calculation summary panel showing total premium, budget, and Ontop/Deduct result. Participates in the `LOAN_TERMS` lockpoint group (locked at Approval HWM).

## Acceptance Criteria
- [ ] Insurance section appears in Loan Setup stage after collateral section
- [ ] Credit Insurance sub-section visible when product type `credit_insurance: enabled`
- [ ] Voluntary/Compulsory sub-section visible when product type `voluntary_insurance: enabled`
- [ ] Calculation summary panel shows total premium, budget, and Ontop/Deduct label
- [ ] All insurance fields belong to `LOAN_TERMS` lockpoint group
- [ ] Locked fields show read-only with lock indicator and tooltip
- [ ] Save Draft persists all insurance data including partial selections

## Dependencies
- Smart Form capability (section registration, lockpoint system)
- Product Type Configuration (insurance enablement flags)
- Credit Insurance Plan Retrieval (populates credit sub-section)
- External Insurance Reference Lookup (populates voluntary sub-section)
- Insurance Premium Ontop/Deduct Calculator (populates calculation summary)

## Notes / Open Questions
- Exact section ordering within Loan Setup (after collateral, before or alongside Finance Page?)

---

## Merge Checklist
*Complete before marking Live and archiving this file.*
- [ ] `FEATURE_insurance-section-smart-form.md` created or updated in `capabilities/insurance-integration/features/`
- [ ] `CAPABILITY.md` feature inventory updated
- [ ] `PRODUCT.md` capability registry updated (if new capability)
- [ ] `BACKLOG.md` row moved to ✅ LIVE
- [ ] `CHANGELOG` entry written
- [ ] This file moved to `backlog/archived/`
