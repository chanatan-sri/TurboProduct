# Feature: Insurance Premium Ontop/Deduct Calculator

**Capability**: [Insurance Integration](../CAPABILITY.md)
**Product**: Onigiri — Loan Origination System
**Owner**: Engineering
**Status**: 💡 Concept
**Created**: 2026-03-30
**Last Updated**: 2026-03-30

---

## User Story

**As a** Credit Officer (CO),
**I want** the system to automatically determine whether the insurance premium is added to the loan amount (Ontop) or deducted from net disbursement (Deduct),
**so that** the loan calculation reflects the correct financial impact without manual computation.

## Job to Be Done

When insurance is selected or changed, the system computes the total insurance premium across all insurance types, compares it against the campaign's insurance budget threshold (`max_credit_line × insurance_budget_pct`), and determines whether the premium is Ontop or Deduct. This result feeds into the Plan Calculation API request to produce the correct installment schedule and net disbursement amount.

---

## Acceptance Criteria

- [ ] AC-1: System reads `insurance_budget_pct` from the active campaign's Pricing configuration (default: 10%).
- [ ] AC-2: System reads `max_credit_line` from the active campaign's Pricing configuration.
- [ ] AC-3: `insurance_budget = max_credit_line × insurance_budget_pct`.
- [ ] AC-4: `total_premium = credit_insurance_premium + SUM(voluntary_compulsory_premiums[])`. If credit insurance is opted out, its contribution is 0.
- [ ] AC-5: If `total_premium ≤ insurance_budget` → method = **Ontop** — premium is added to total loan amount.
- [ ] AC-6: If `total_premium > insurance_budget` → method = **Deduct** — premium is subtracted from net disbursement amount.
- [ ] AC-7: If `total_premium == 0` → method = **none** — no insurance impact on loan calculation.
- [ ] AC-8: Calculation result (`total_premium`, `insurance_budget`, `method`, `calculated_at`) is stored in `insurance.calculation` in the application JSON.
- [ ] AC-9: Recalculation triggers on every insurance selection change: plan selection, plan opt-out, reference confirmation, reference removal.
- [ ] AC-10: Updated calculation result is displayed to CO showing: total premium, budget threshold, and resulting method (Ontop/Deduct) with clear Thai-language label.
- [ ] AC-11: Calculation triggers Plan Calculation API recalculation (delegated to Feature 5).

---

## Edge Cases and Error States

| Scenario | Behavior |
|---|---|
| Campaign has no `insurance_budget_pct` configured | Use system default of 10% |
| `max_credit_line` is 0 or null | Insurance budget = 0; any premium > 0 results in **Deduct** |
| Only credit insurance selected (no voluntary) | Total = credit premium only |
| Only voluntary selected (credit opted out) | Total = sum of voluntary premiums only |
| Premium exactly equals budget | **Ontop** (≤ condition) |
| CO removes all insurance (total = 0) | Method = **none**; Plan Calc reverts to no-insurance calculation |

---

## Dependencies

- Campaign Configuration — `insurance_budget_pct` and `max_credit_line` parameters in Pricing dimension
- Credit Insurance Plan Retrieval (Feature 1) — provides credit insurance premium
- External Insurance Reference Lookup (Feature 2) — provides voluntary/compulsory premiums
- Insurance Data Pass-through to Plan Calculation (Feature 5) — consumes the Ontop/Deduct result
