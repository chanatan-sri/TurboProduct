# Feature: Insurance Section in Smart Form

**Capability**: [Insurance Integration](../CAPABILITY.md)
**Product**: Onigiri — Loan Origination System
**Owner**: Engineering
**Status**: 💡 Concept
**Created**: 2026-03-30
**Last Updated**: 2026-03-30

---

## User Story

**As a** Credit Officer (CO),
**I want** a dedicated insurance section within the Loan Setup stage of the Smart Form,
**so that** I can select credit insurance and enter voluntary/compulsory insurance references in one place during loan application entry.

## Job to Be Done

The insurance section is a new section in the Smart Form's Loan Setup stage, positioned after collateral and alongside the Finance Page. It contains two conditionally visible sub-sections: Credit Insurance (plan picker) and Voluntary/Compulsory Insurance (reference number input with multi-reference support). The section participates in the Smart Form's field lockpoint system.

---

## Acceptance Criteria

- [ ] AC-1: Insurance section appears in the Loan Setup stage, after the collateral section.
- [ ] AC-2: Section visibility is controlled by Product Type Configuration: hidden if both `credit_insurance` and `voluntary_insurance` are `disabled`.
- [ ] AC-3: **Credit Insurance sub-section** contains: plan selection dropdown/list (populated by Feature 1), selected plan summary card (name, premium, coverage, insurer), "ไม่ต้องการประกันสินเชื่อ" (No insurance) opt-out option.
- [ ] AC-4: **Voluntary/Compulsory Insurance sub-section** contains: reference number text input with "ค้นหา" (Look up) button, confirmed references list (each showing: plan name, premium, coverage, expiry, insurer, with "ลบ" (Remove) action), "เพิ่มอ้างอิง" (Add reference) button for adding more references.
- [ ] AC-5: **Calculation summary panel** at the bottom of the insurance section displays: total insurance premium, insurance budget (from campaign), Ontop/Deduct result with clear Thai label ("เพิ่มยอดสินเชื่อ" for Ontop / "หักจากยอดจ่ายสุทธิ" for Deduct).
- [ ] AC-6: All insurance fields belong to the `LOAN_TERMS` lockpoint group — locked when HWM ≥ `Approval`.
- [ ] AC-7: When locked, all insurance fields render as read-only with lock indicator and tooltip: "ข้อมูลประกันถูกล็อคเมื่อใบคำขอได้รับอนุมัติแล้ว หากต้องการเปลี่ยนแปลง กรุณายกเลิกและสร้างใบคำขอใหม่".
- [ ] AC-8: Insurance data is persisted in the application JSON under the `insurance` key (see CAPABILITY.md for schema).
- [ ] AC-9: Save Draft action persists all insurance data including partial selections.

---

## Section Field Definitions

### Credit Insurance Sub-section

| Field | Type | Required | Description |
|---|---|---|---|
| `credit_insurance.selected` | boolean | Yes | Whether CO selected a plan (true) or opted out (false) |
| `credit_insurance.plan_id` | string | If selected | Selected plan identifier from API |
| `credit_insurance.plan_name` | string | If selected | Display name of selected plan |
| `credit_insurance.premium` | decimal | If selected | Premium amount |
| `credit_insurance.coverage_amount` | decimal | If selected | Coverage amount |
| `credit_insurance.coverage_term` | integer | If selected | Coverage term in months |
| `credit_insurance.insurer_name` | string | If selected | Insurer display name |
| `credit_insurance.source` | string | System | Always `"insurance_api"` |
| `credit_insurance.fetched_at` | datetime | System | Timestamp of API response |

### Voluntary/Compulsory Insurance Sub-section

| Field (per reference) | Type | Required | Description |
|---|---|---|---|
| `voluntary_compulsory[].reference_number` | string | Yes | Reference number from external purchase |
| `voluntary_compulsory[].plan_name` | string | From API | Policy plan name |
| `voluntary_compulsory[].premium` | decimal | From API | Premium amount |
| `voluntary_compulsory[].coverage_amount` | decimal | From API | Coverage amount |
| `voluntary_compulsory[].coverage_term` | integer | From API | Coverage term in months |
| `voluntary_compulsory[].expiry_date` | date | From API | Policy expiry date |
| `voluntary_compulsory[].insurer_name` | string | From API | Insurer display name |
| `voluntary_compulsory[].policy_status` | string | From API | Must be `"active"` to confirm |
| `voluntary_compulsory[].source` | string | System | Always `"insurance_api_ref"` |
| `voluntary_compulsory[].confirmed_at` | datetime | System | Timestamp of CO confirmation |

### Calculation Summary

| Field | Type | Description |
|---|---|---|
| `calculation.total_premium` | decimal | Sum of all premiums |
| `calculation.insurance_budget` | decimal | `max_credit_line × insurance_budget_pct` |
| `calculation.method` | enum | `ontop` / `deduct` / `none` |
| `calculation.calculated_at` | datetime | Last calculation timestamp |

---

## Edge Cases and Error States

| Scenario | Behavior |
|---|---|
| Product type has no insurance flags configured | Insurance section is hidden (default: both disabled) |
| Only credit insurance enabled | Voluntary/Compulsory sub-section hidden; only Credit Insurance sub-section shown |
| Only voluntary insurance enabled | Credit Insurance sub-section hidden; only Voluntary/Compulsory sub-section shown |
| CO saves draft with partial insurance data | Persisted as-is; incomplete selections do not block save |
| CO submits application with no insurance selected | Valid — insurance is optional; `total_premium = 0`, `method = "none"` |

---

## Dependencies

- Smart Form capability — section registration in Loan Setup stage, lockpoint group membership
- Product Type Configuration — `credit_insurance` and `voluntary_insurance` enablement flags
- Credit Insurance Plan Retrieval (Feature 1) — populates Credit Insurance sub-section
- External Insurance Reference Lookup (Feature 2) — populates Voluntary/Compulsory sub-section
- Insurance Premium Ontop/Deduct Calculator (Feature 3) — populates calculation summary panel
