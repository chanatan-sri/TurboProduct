# Feature: Credit Insurance Plan Retrieval

**Capability**: [Insurance Integration](../CAPABILITY.md)
**Product**: Onigiri — Loan Origination System
**Owner**: Engineering
**Status**: 💡 Concept
**Created**: 2026-03-30
**Last Updated**: 2026-03-30

---

## User Story

**As a** Credit Officer (CO),
**I want to** see a list of eligible credit insurance plans fetched from the external insurance system when I reach the insurance section,
**so that** I can select an appropriate insurance plan for the customer's loan — or opt out if insurance is not needed.

## Job to Be Done

When the CO navigates to the Credit Insurance sub-section during Loan Setup, the system automatically calls the external insurance API with the current loan context and displays the available plans. The CO selects one plan (or explicitly opts out). The selected plan's premium feeds into the Ontop/Deduct calculator.

---

## Acceptance Criteria

- [ ] AC-1: When CO enters the Credit Insurance sub-section, system calls `POST /insurance/credit/plans` with `{ loan_amount, customer_id, collateral_type, campaign_id }`.
- [ ] AC-2: On successful API response, system displays a selectable list of eligible plans showing: plan name, premium amount, coverage amount, coverage term, insurer name.
- [ ] AC-3: CO can select exactly one plan from the list.
- [ ] AC-4: CO can explicitly opt out by selecting "ไม่ต้องการประกันสินเชื่อ" (No credit insurance) option.
- [ ] AC-5: Selected plan data is stored in the application JSON under `insurance.credit_insurance` with `source: "insurance_api"` and `fetched_at` timestamp.
- [ ] AC-6: When CO opts out, `insurance.credit_insurance.selected` is set to `false` and no premium is contributed to the total.
- [ ] AC-7: On API error or timeout, system displays error message with retry option. CO can proceed without credit insurance if API is unavailable.
- [ ] AC-8: Changing the selected plan updates `insurance.credit_insurance` and triggers Ontop/Deduct recalculation.
- [ ] AC-9: Credit Insurance sub-section is only visible when the product type has `credit_insurance: enabled`.

---

## Edge Cases and Error States

| Scenario | Behavior |
|---|---|
| API returns empty plans list | Display message: "ไม่มีแผนประกันสินเชื่อที่ตรงเงื่อนไข" — CO proceeds without credit insurance |
| API timeout (> 3 seconds) | Show timeout error with retry button |
| API returns 5xx | Show system error with retry button; CO can skip |
| CO changes loan amount after selecting plan | Plan selection is preserved; Ontop/Deduct recalculates with existing premium. Open question: should plan list be re-fetched? |
| Insurance section locked (HWM ≥ Approval) | Plan selection displays as read-only with lock indicator |

---

## Dependencies

- External Insurance System API (`POST /insurance/credit/plans`)
- Insurance Section in Smart Form (Feature 4) — provides the rendering container
- Insurance Premium Ontop/Deduct Calculator (Feature 3) — consumes selected premium
- Product Type Configuration — `credit_insurance` enablement flag
