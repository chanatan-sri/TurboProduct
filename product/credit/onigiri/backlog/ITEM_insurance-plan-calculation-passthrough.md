# ITEM: Insurance Data Pass-through to Plan Calculation

**Status:** Concept
**Capability:** Insurance Integration
**Product:** Onigiri
**Owner:** TBD
**Created:** 2026-03-30
**Last Updated:** 2026-03-30

---

## What & Why
Include insurance premium and Ontop/Deduct designation in the Plan Calculation API request so the installment schedule on the Finance Page reflects the insurance impact. Any insurance change triggers recalculation. This ensures the CO sees accurate loan terms — including insurance-adjusted amounts — before submitting the application.

## Acceptance Criteria
- [ ] Plan Calculation API request includes `insurance_total_premium`, `insurance_method`, and `insurance_items[]`
- [ ] Ontop: premium added to loan amount in calculation; Deduct: premium subtracted from net disbursement
- [ ] Any insurance change triggers Plan Calculation API recalculation
- [ ] Finance Page displays updated installment schedule with insurance impact
- [ ] CO cannot proceed to Summary until valid Plan Calculation response received
- [ ] Plan Calc failure shows error with retry option

## Dependencies
- Insurance Premium Ontop/Deduct Calculator (provides total_premium and method)
- Plan Calculation API (external service)
- Smart Form Finance Page (displays results)

## Notes / Open Questions
- Handling when insurance premium causes loan amount to exceed campaign maximum

---

## Merge Checklist
*Complete before marking Live and archiving this file.*
- [ ] `FEATURE_insurance-plan-calculation-passthrough.md` created or updated in `capabilities/insurance-integration/features/`
- [ ] `CAPABILITY.md` feature inventory updated
- [ ] `PRODUCT.md` capability registry updated (if new capability)
- [ ] `BACKLOG.md` row moved to ✅ LIVE
- [ ] `CHANGELOG` entry written
- [ ] This file moved to `backlog/archived/`
