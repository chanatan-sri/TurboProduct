# Feature: DipChip Integration Gate

**Parent Capability**: AI-Assisted Data Entry — [CAPABILITY](../CAPABILITY.md)
**Product**: Onigiri — [PRODUCT](../../../PRODUCT.md)
**Engineering Owner**: TBD
**Status**: Concept
**Last Updated**: 2026-03-30

---

## User Story

As a **Credit Officer**, I want the system to require a successful DipChip ID card chip read before showing document upload options, so that baseline identity data is always captured from the authoritative chip source first.

## Job-to-be-Done

DipChip reads the government-issued chip in a Thai national ID card. This is the most authoritative source for identity data (name, ID number, date of birth, address). By requiring DipChip first, we ensure:
1. The customer's identity is verified via chip before any document processing begins.
2. Identity fields are populated from the authoritative source, preventing AI extraction from overwriting them.
3. Upload boxes only appear after a successful chip read — enforcing the correct sequence.

---

## Acceptance Criteria

| # | Criterion | Pass Condition |
|---|-----------|---------------|
| AC-1 | Upload boxes hidden before DipChip | Document upload UI is not visible/disabled until DipChip read succeeds |
| AC-2 | DipChip populates identity fields | Name, ID number, date of birth, address written to Smart Form identity section |
| AC-3 | Source attribution | Each DipChip-populated field tagged with `source: "dipchip"` in the application JSON |
| AC-4 | Upload boxes appear after success | After successful DipChip read, all configured upload boxes for the product type become visible |
| AC-5 | DipChip is mandatory | Cannot be skipped — no bypass path to document upload without chip read |
| AC-6 | Failed chip read | Show error message; CO can retry. Upload boxes remain hidden. |
| AC-7 | DipChip fields are editable | CO can still manually edit DipChip-populated fields (tagged as `manual_override` on edit) |

---

## Edge Cases & Error States

| Scenario | Expected Behavior |
|----------|------------------|
| DipChip hardware not connected | Error: "ไม่พบเครื่องอ่านบัตร กรุณาเชื่อมต่อ" — upload boxes remain hidden |
| Chip read timeout | Error: "อ่านบัตรไม่สำเร็จ กรุณาลองใหม่" — CO retries |
| Card is foreign passport (no chip) | DipChip gate does not apply to foreigner identity variant — separate flow TBD |
| CO edits a DipChip field | Field source changes from `dipchip` to `manual_override`; original DipChip value preserved in audit trail |

---

## Dependencies

| Dependency | Type | Notes |
|-----------|------|-------|
| DipChip hardware integration | External | Assumes existing or parallel DipChip reader integration |
| Smart Form identity section | Internal — [CAPABILITY](../../smart-form/CAPABILITY.md) | Target fields for DipChip data |
| Source attribution model | Internal | Application JSON must support `source` metadata per field |
