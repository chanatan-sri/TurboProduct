# Feature: Extraction Confidence Display

**Parent Capability**: AI-Assisted Data Entry — [CAPABILITY](../CAPABILITY.md)
**Product**: Onigiri — [PRODUCT](../../../PRODUCT.md)
**Engineering Owner**: TBD
**Status**: Concept
**Last Updated**: 2026-03-30

---

## User Story

As a **Credit Officer**, I want to see confidence indicators on AI-extracted fields, so that I know which values to double-check and which I can trust.

## Job-to-be-Done

Not all AI-extracted values have the same reliability. ID numbers are typically extracted with high confidence; Thai addresses are harder. This feature provides visual cues so the CO can focus their review effort on low-confidence fields rather than re-checking everything.

---

## Confidence Display Rules

| Confidence Range | Fill Behavior | Visual Indicator | Tooltip (TH) |
|-----------------|--------------|------------------|---------------|
| >= threshold | Auto-fill | 🟢 Green AI icon | "ข้อมูลจาก AI (ความมั่นใจสูง)" |
| >= 0.70 and < threshold | Auto-fill with warning | 🟡 Yellow AI icon | "ข้อมูลจาก AI (ความมั่นใจต่ำ — กรุณาตรวจสอบ)" |
| < 0.70 | Leave blank | ⚪ Gray AI icon | "AI ไม่สามารถอ่านข้อมูลนี้ได้อย่างมั่นใจ" |
| `not_extractable` | Leave blank | No indicator | — |
| `source: "dipchip"` | Filled by DipChip | 🔵 Chip icon | "ข้อมูลจากชิปบัตรประชาชน" |
| `source: "manual_override"` | CO edited | No special icon | — |

---

## Acceptance Criteria

| # | Criterion | Pass Condition |
|---|-----------|---------------|
| AC-1 | Green indicator | Fields with confidence >= threshold show green AI icon |
| AC-2 | Yellow indicator | Fields with confidence >= 0.70 and < threshold show yellow AI icon with "please verify" message |
| AC-3 | Gray indicator | Fields with confidence < 0.70 show gray AI icon on empty field |
| AC-4 | No indicator for not_extractable | Fields that Wasabi could not extract show no AI indicator |
| AC-5 | DipChip indicator | DipChip-sourced fields show chip icon (distinct from AI icon) |
| AC-6 | Indicator clears on manual edit | When CO edits a pre-filled field, AI indicator is removed (becomes manual_override) |
| AC-7 | Tooltip with confidence % | Hovering AI icon shows confidence percentage and source document name |
| AC-8 | Per-field threshold | Different fields can have different thresholds (configured in mapping registry) |
| AC-9 | Section-level summary | Each Smart Form section header shows count of AI-filled fields and count needing review |

---

## Edge Cases & Error States

| Scenario | Expected Behavior |
|----------|------------------|
| Confidence exactly at threshold | Treated as >= threshold (green indicator) |
| Confidence exactly at 0.70 | Treated as >= 0.70 (yellow indicator, not blank) |
| Field filled by AI, then CO edits, then reverts to AI value | AI indicator does not return — field remains `manual_override` |
| Multiple AI extractions update same field | Indicator shows latest extraction's confidence |

---

## Dependencies

| Dependency | Type | Notes |
|-----------|------|-------|
| Smart Form Pre-fill Engine | Internal — [FEATURE](FEATURE_smart-form-prefill-engine.md) | Provides source + confidence metadata per field |
| Extraction Field Mapping Registry | Internal — [FEATURE](FEATURE_extraction-field-mapping-registry.md) | Provides per-field confidence thresholds |
| Smart Form UI | Internal — [CAPABILITY](../../smart-form/CAPABILITY.md) | UI must render indicators and tooltips |
