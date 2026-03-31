# ITEM: Credit Insurance Plan Retrieval

**Status:** Concept
**Capability:** Insurance Integration
**Product:** Onigiri
**Owner:** TBD
**Created:** 2026-03-30
**Last Updated:** 2026-03-30

---

## What & Why
Retrieve eligible credit insurance plans from the external insurance API when the CO enters the insurance section during Loan Setup. The CO selects a plan or opts out. The selected premium feeds into the Ontop/Deduct calculator to determine loan calculation impact. This enables bundling credit insurance into loan origination without requiring the CO to use a separate system.

## Acceptance Criteria
- [ ] System calls `POST /insurance/credit/plans` with loan context on section entry
- [ ] Eligible plans displayed as a selectable list with plan name, premium, coverage, insurer
- [ ] CO can select one plan or opt out
- [ ] Selected plan stored in application JSON with `source: "insurance_api"`
- [ ] API failure shows retry option; CO can proceed without insurance

## Dependencies
- External Insurance System API
- Insurance Section in Smart Form
- Insurance Premium Ontop/Deduct Calculator
- Product Type Configuration (`credit_insurance` flag)

## Notes / Open Questions
- Should plan list be re-fetched when loan amount changes?
- API authentication mechanism TBD

---

## Merge Checklist
*Complete before marking Live and archiving this file.*
- [ ] `FEATURE_credit-insurance-plan-retrieval.md` created or updated in `capabilities/insurance-integration/features/`
- [ ] `CAPABILITY.md` feature inventory updated
- [ ] `PRODUCT.md` capability registry updated (if new capability)
- [ ] `BACKLOG.md` row moved to ✅ LIVE
- [ ] `CHANGELOG` entry written
- [ ] This file moved to `backlog/archived/`
