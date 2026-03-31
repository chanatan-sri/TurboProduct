# Feature: Insurance Data Pass-through to Plan Calculation

**Capability**: [Insurance Integration](../CAPABILITY.md)
**Product**: Onigiri — Loan Origination System
**Owner**: Engineering
**Status**: 💡 Concept
**Created**: 2026-03-30
**Last Updated**: 2026-03-30

---

## User Story

**As a** Credit Officer (CO),
**I want** the installment schedule on the Finance Page to automatically reflect the insurance premium impact (Ontop or Deduct),
**so that** I can see the accurate loan terms before submitting the application.

## Job to Be Done

When insurance data changes (plan selected, reference confirmed/removed, opt-out), the system includes the insurance premium and Ontop/Deduct designation in the Plan Calculation API request. The API recalculates the installment schedule with the insurance impact factored in. The updated schedule is displayed on the Finance Page. This reuses the same recalculation pattern as the restructure Finance Page flow.

---

## Acceptance Criteria

- [ ] AC-1: Plan Calculation API request includes three new fields: `insurance_total_premium` (decimal), `insurance_method` (enum: `ontop` / `deduct` / `none`), `insurance_items[]` (array of `{ type, plan_id, plan_name, premium, source }`).
- [ ] AC-2: When `insurance_method = "ontop"`, the Plan Calculation API adds `insurance_total_premium` to the base loan amount before computing installments.
- [ ] AC-3: When `insurance_method = "deduct"`, the Plan Calculation API subtracts `insurance_total_premium` from net disbursement without changing the loan amount.
- [ ] AC-4: When `insurance_method = "none"`, insurance fields are ignored in calculation.
- [ ] AC-5: Any insurance change triggers Plan Calculation API recalculation — same trigger pattern as campaign/plan option change on the Finance Page.
- [ ] AC-6: Finance Page displays updated installment schedule reflecting insurance impact: adjusted loan amount (if Ontop), adjusted net disbursement (if Deduct), insurance premium line item in summary.
- [ ] AC-7: CO cannot progress from Loan Setup to Summary until Plan Calculation API returns a valid response with insurance factored in.
- [ ] AC-8: If Plan Calculation API fails after insurance change, show error with retry — do not allow submission with stale calculation.

---

## Edge Cases and Error States

| Scenario | Behavior |
|---|---|
| Insurance changes while Plan Calc is in-flight | Cancel previous request; send new request with latest insurance data |
| Plan Calc API timeout after insurance change | Show timeout error; CO retries; cannot proceed to Summary |
| CO removes all insurance (method = none) | Trigger recalculation without insurance; Finance Page reverts to base calculation |
| Insurance premium causes loan amount to exceed campaign max | Plan Calc API returns validation error; CO must adjust insurance selection or loan amount |

---

## Dependencies

- Insurance Premium Ontop/Deduct Calculator (Feature 3) — provides `total_premium` and `method`
- Plan Calculation API (external service) — performs actual computation
- Smart Form Finance Page — displays the updated installment schedule
- Insurance Section in Smart Form (Feature 4) — triggers recalculation on insurance change
