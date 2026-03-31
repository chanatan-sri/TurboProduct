# ITEM: Insurance Premium Ontop/Deduct Calculator

**Status:** Concept
**Capability:** Insurance Integration
**Product:** Onigiri
**Owner:** TBD
**Created:** 2026-03-30
**Last Updated:** 2026-03-30

---

## What & Why
Core business logic that determines whether insurance premium is added to the loan amount (Ontop) or deducted from net disbursement (Deduct). The decision is based on comparing total insurance premium against the campaign's insurance budget (`max_credit_line × insurance_budget_pct`). This ensures loan calculations correctly reflect the financial impact of insurance on the customer's borrowing and payout amounts.

## Acceptance Criteria
- [ ] `insurance_budget = max_credit_line × insurance_budget_pct` (from campaign Pricing config)
- [ ] `total_premium = credit_insurance_premium + SUM(voluntary_compulsory_premiums[])`
- [ ] If `total_premium ≤ insurance_budget` → Ontop (add to loan amount)
- [ ] If `total_premium > insurance_budget` → Deduct (subtract from net disbursement)
- [ ] If `total_premium == 0` → No effect
- [ ] Recalculates on every insurance selection change
- [ ] Result displayed to CO with clear Thai-language label

## Dependencies
- Campaign Configuration (`insurance_budget_pct`, `max_credit_line`)
- Credit Insurance Plan Retrieval (provides credit premium)
- External Insurance Reference Lookup (provides voluntary/compulsory premiums)
- Insurance Data Pass-through to Plan Calculation (consumes result)

## Notes / Open Questions
- Default `insurance_budget_pct` is 10% when campaign does not specify

---

## Merge Checklist
*Complete before marking Live and archiving this file.*
- [ ] `FEATURE_insurance-premium-ontop-deduct-calculator.md` created or updated in `capabilities/insurance-integration/features/`
- [ ] `CAPABILITY.md` feature inventory updated
- [ ] `PRODUCT.md` capability registry updated (if new capability)
- [ ] `BACKLOG.md` row moved to ✅ LIVE
- [ ] `CHANGELOG` entry written
- [ ] This file moved to `backlog/archived/`
