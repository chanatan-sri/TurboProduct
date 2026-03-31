# ITEM: Campaign Insurance Budget Configuration

**Status:** Concept
**Capability:** Loan Campaign Configuration
**Product:** Onigiri
**Owner:** TBD
**Created:** 2026-03-30
**Last Updated:** 2026-03-30

---

## What & Why
Add `insurance_budget_pct` parameter to the Campaign Configuration's Pricing dimension. This configurable percentage (default 10%) of `max_credit_line` determines the insurance budget threshold used by the Ontop/Deduct calculator. Each campaign can set its own insurance budget percentage, allowing different loan products to have different insurance tolerance levels.

## Acceptance Criteria
- [ ] `insurance_budget_pct` field added to Campaign Pricing Configuration (decimal, default 10%)
- [ ] Validated: must be between 0% and 100%
- [ ] Campaign Builder UI includes the field in the Pricing section
- [ ] Existing campaigns without the field default to 10%
- [ ] Value is readable by the Insurance Premium Ontop/Deduct Calculator at runtime

## Dependencies
- Loan Campaign Configuration capability (Pricing Configuration feature)
- Insurance Premium Ontop/Deduct Calculator (consumes this parameter)

## Notes / Open Questions
- `max_credit_line` already exists in the Pricing Parameters table — confirm it maps to the correct field

---

## Merge Checklist
*Complete before marking Live and archiving this file.*
- [ ] `FEATURE_pricing-configuration.md` updated in `capabilities/loan-campaign-configuration/features/` (or Pricing Configuration spec)
- [ ] `CAPABILITY.md` feature inventory updated
- [ ] `PRODUCT.md` capability registry updated (if new capability)
- [ ] `BACKLOG.md` row moved to ✅ LIVE
- [ ] `CHANGELOG` entry written
- [ ] This file moved to `backlog/archived/`
