# Feature: External Insurance Reference Lookup

**Capability**: [Insurance Integration](../CAPABILITY.md)
**Product**: Onigiri — Loan Origination System
**Owner**: Engineering
**Status**: 💡 Concept
**Created**: 2026-03-30
**Last Updated**: 2026-03-30

---

## User Story

**As a** Credit Officer (CO),
**I want to** enter a reference number from a customer's externally purchased insurance policy and see the policy details retrieved from the insurance system,
**so that** I can verify the insurance is valid and include its premium in the loan calculation.

## Job to Be Done

When a customer has purchased voluntary or compulsory insurance from an external website, they receive a reference number. The CO enters this reference number into Onigiri's Voluntary/Compulsory Insurance sub-section. The system calls the external insurance API to validate the reference and retrieve policy details. If the policy is active, the CO confirms it and the premium is added to the total insurance calculation. The CO can add multiple references for different insurance policies.

---

## Acceptance Criteria

- [ ] AC-1: CO can enter a reference number in the Voluntary/Compulsory Insurance sub-section and click "ค้นหา" (Look up).
- [ ] AC-2: System calls `GET /insurance/reference/{ref_number}` with the entered reference number.
- [ ] AC-3: On successful response with `policy_status: "active"`, system displays: plan name, premium, coverage amount, coverage term, expiry date, insurer name.
- [ ] AC-4: CO clicks "ยืนยัน" (Confirm) to add the insurance to the application.
- [ ] AC-5: Confirmed insurance data is stored in `insurance.voluntary_compulsory[]` array with `source: "insurance_api_ref"` and `confirmed_at` timestamp.
- [ ] AC-6: CO can add multiple references — each confirmed reference appends to the `voluntary_compulsory[]` array.
- [ ] AC-7: CO can remove a previously confirmed reference (before lockpoint). Removal recalculates total premium.
- [ ] AC-8: On `policy_status: "expired"`, system shows error: "กรมธรรม์หมดอายุแล้ว" — reference is not added.
- [ ] AC-9: On `policy_status: "cancelled"`, system shows error: "กรมธรรม์ถูกยกเลิกแล้ว" — reference is not added.
- [ ] AC-10: On 404 Not Found, system shows error: "ไม่พบหมายเลขอ้างอิง กรุณาตรวจสอบ" — CO re-enters.
- [ ] AC-11: On 5xx / timeout, system shows error: "ระบบไม่สามารถตรวจสอบได้ กรุณาลองใหม่" — CO retries.
- [ ] AC-12: Duplicate reference number detection — if CO enters a reference already confirmed in this application, show warning: "หมายเลขอ้างอิงนี้ถูกเพิ่มแล้ว".
- [ ] AC-13: Each confirmed/removed reference triggers Ontop/Deduct recalculation.
- [ ] AC-14: Voluntary/Compulsory sub-section is only visible when product type has `voluntary_insurance: enabled`.

---

## Edge Cases and Error States

| Scenario | Behavior |
|---|---|
| Reference number has leading/trailing whitespace | Trim before API call |
| Same reference number entered twice | Block with duplicate warning (AC-12) |
| API returns unexpected `policy_status` value | Treat as error — show generic message, do not add |
| CO removes all voluntary references | `voluntary_compulsory[]` becomes empty array; total premium recalculates |
| Insurance section locked (HWM ≥ Approval) | All confirmed references display as read-only; no add/remove actions available |

---

## Dependencies

- External Insurance System API (`GET /insurance/reference/{ref_number}`)
- Insurance Section in Smart Form (Feature 4) — provides the rendering container
- Insurance Premium Ontop/Deduct Calculator (Feature 3) — consumes confirmed premiums
- Product Type Configuration — `voluntary_insurance` enablement flag
