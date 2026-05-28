# Feature: Pre-fill Conflict Resolution

**Parent Capability**: AI-Assisted Data Entry — [CAPABILITY](../CAPABILITY.md)
**Product**: Onigiri — [PRODUCT](../../../PRODUCT.md)
**Engineering Owner**: TBD
**Status**: Concept
**Last Updated**: 2026-03-30

---

## User Story

As a **Credit Officer**, I want the system to intelligently handle conflicts when multiple data sources provide values for the same field, so that the most reliable data is used by default and I understand where each value came from.

## Job-to-be-Done

Multiple data sources can populate the same Smart Form field: DipChip, AI extraction (from one or more documents), manual entry, and DaVinci pre-fill. This feature defines the priority rules and conflict resolution behavior so the CO always sees the most authoritative value.

---

## Source Priority Hierarchy

| Priority | Source Tag | Description | Overwritable By |
|:--------:|-----------|-------------|-----------------|
| 1 | `dipchip` | Government chip data — authoritative for identity | `manual_override` only |
| 2 | `manual_override` | CO intentionally corrected a value | Nothing (highest user intent) |
| 3 | `manual_entry` | CO typed the value from scratch | `manual_override` |
| 4 | `ai_extraction` | Extracted from document image by Wasabi | `dipchip`, `manual_override`, `manual_entry` |
| 5 | `prefill_davinci` | Pre-filled from DaVinci master data (restructure) | All of the above |

---

## Conflict Resolution Rules

| Scenario | Resolution | Rationale |
|----------|-----------|-----------|
| DipChip provides X, then AI extraction also provides X | **DipChip wins** — AI does not overwrite | Chip data is authoritative for identity |
| AI extraction from doc A provides X, then AI from doc B also provides X | **Later extraction overwrites** with notification | CO uploaded a second document; latest is intentional |
| Manual entry exists, then AI extraction provides same field | **Manual entry preserved** — AI does not overwrite | CO intentionally typed the value |
| AI provides X, then CO edits to Y | Source changes to `manual_override` | CO's correction is highest user intent |
| AI provides X, then DipChip provides X (unlikely order) | **DipChip wins** — overwrites AI value | DipChip is mandatory and authoritative |

---

## Acceptance Criteria

| # | Criterion | Pass Condition |
|---|-----------|---------------|
| AC-1 | Priority enforcement | AI extraction never overwrites DipChip, manual_override, or manual_entry values |
| AC-2 | Source tag updated | When a value is overwritten by a higher-priority source, the source tag is updated |
| AC-3 | Previous value preserved | When AI extraction from doc B overwrites doc A's value, the previous value + source is stored in field metadata for audit |
| AC-4 | Notification on same-priority conflict | When two AI extractions target the same field, CO sees: "ข้อมูล [field] ถูกอัปเดตจาก [doc B] (ค่าเดิมจาก [doc A]: [old value])" |
| AC-5 | Manual override tagging | When CO edits any pre-filled field, source changes to `manual_override` regardless of original source |
| AC-6 | No silent overwrites | Fields from higher-priority sources are never silently overwritten — if an AI extraction attempts to overwrite a DipChip value, the AI value is discarded without notification (DipChip is expected to be authoritative) |

---

## Edge Cases & Error States

| Scenario | Expected Behavior |
|----------|------------------|
| DipChip provides empty value for a field, AI extracts a value | AI value written (empty DipChip = no value, not a higher-priority value) |
| CO clears a pre-filled field (sets to empty) | Field tagged `manual_override` with empty value — AI cannot re-fill |
| Restructure app: DaVinci pre-fills, then AI extracts different value | AI wins (priority 4 > priority 5); CO notified of change |
| Same document uploaded twice (re-upload) | Second extraction replaces first — same source, same priority |

---

## Dependencies

| Dependency | Type | Notes |
|-----------|------|-------|
| Smart Form Pre-fill Engine | Internal — [FEATURE](FEATURE_smart-form-prefill-engine.md) | Pre-fill Engine calls Conflict Resolution before writing |
| DipChip Integration Gate | Internal — [FEATURE](FEATURE_dipchip-integration-gate.md) | DipChip values must be tagged before AI extraction runs |
| Source attribution model | Internal | Application JSON must support per-field `source` + `previous_value` metadata |
