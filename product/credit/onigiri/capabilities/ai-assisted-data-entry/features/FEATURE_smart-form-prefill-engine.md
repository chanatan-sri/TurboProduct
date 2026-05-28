# Feature: Smart Form Pre-fill Engine

**Parent Capability**: AI-Assisted Data Entry — [CAPABILITY](../CAPABILITY.md)
**Product**: Onigiri — [PRODUCT](../../../PRODUCT.md)
**Engineering Owner**: TBD
**Status**: Concept
**Last Updated**: 2026-03-30

---

## User Story

As a **Credit Officer**, I want extracted document data to automatically fill the relevant Smart Form fields, so that I spend less time on manual data entry and reduce transcription errors.

## Job-to-be-Done

The Pre-fill Engine is the core runtime component that:
1. Receives extracted fields from a Wasabi `DocumentExtractionReport`
2. Resolves target Smart Form fields using the Extraction Field Mapping Registry
3. Applies confidence thresholds to decide: auto-fill / fill-with-warning / leave-blank
4. Writes values to the application JSON with source attribution metadata
5. Triggers UI re-render of affected fields with appropriate visual indicators

---

## Acceptance Criteria

| # | Criterion | Pass Condition |
|---|-----------|---------------|
| AC-1 | Field resolution | Each `check_name` in the extraction report is resolved to a Smart Form `field_path` via the mapping registry |
| AC-2 | Confidence-based fill | Fields filled based on confidence vs. threshold: >= threshold → auto-fill; >= 0.70 → fill with warning; < 0.70 → leave blank |
| AC-3 | Source attribution | Every pre-filled field tagged with `source: "ai_extraction"`, `document_id`, `confidence`, `extraction_timestamp` |
| AC-4 | Visual distinction | Pre-filled fields are visually distinct from manually entered fields (e.g., AI icon indicator) |
| AC-5 | All values editable | CO can override any pre-filled value; overridden values tagged `source: "manual_override"` |
| AC-6 | Cross-section fill | Extracted values can fill fields in sections the CO has not yet visited (values ready when CO navigates there) |
| AC-7 | Persist on save | Pre-filled values written to in-memory application JSON; persisted to DocumentDB on next save-draft |
| AC-8 | Unmapped fields ignored | Extracted fields with no mapping in the registry are silently ignored |
| AC-9 | Transform applied | If `value_transform_inbound` is defined for a mapping, transformation applied before writing |
| AC-10 | Locked fields respected | Fields locked by HWM (state_high_water_mark) are not overwritten by pre-fill |

---

## Pre-fill Write Logic

```
FOR each extracted_field in DocumentExtractionReport.extracted_fields:
    mapping = lookup(extraction_template, extracted_field.check_name)
    IF mapping is NULL OR mapping.extraction_enabled = false:
        SKIP (log unmapped/disabled field)

    target_field = resolve(mapping.field_path, active_variant)
    IF target_field does not exist in current form:
        SKIP (log missing field)

    IF target_field is locked by HWM:
        SKIP (field is read-only)

    existing_source = target_field.metadata.source
    IF existing_source IN ["dipchip", "manual_override", "manual_entry"]:
        SKIP (higher priority source — do not overwrite)

    IF extracted_field.status = "not_extractable":
        SKIP

    IF extracted_field.confidence < 0.70:
        SET target_field.metadata = { source: "ai_extraction", confidence: extracted_field.confidence, status: "below_threshold", document_id }
        DO NOT write value (leave blank, show gray indicator)

    ELSE IF extracted_field.confidence < mapping.confidence_threshold:
        value = apply_transform(extracted_field.extracted_value, mapping.value_transform_inbound)
        WRITE target_field.value = value
        SET target_field.metadata = { source: "ai_extraction", confidence: extracted_field.confidence, status: "low_confidence", document_id }
        SHOW yellow warning indicator

    ELSE:
        value = apply_transform(extracted_field.extracted_value, mapping.value_transform_inbound)
        WRITE target_field.value = value
        SET target_field.metadata = { source: "ai_extraction", confidence: extracted_field.confidence, status: "confident", document_id }
        SHOW green indicator
```

---

## Edge Cases & Error States

| Scenario | Expected Behavior |
|----------|------------------|
| Extraction returns 0 fields | No fields pre-filled. Info message: "ไม่สามารถอ่านข้อมูลจากเอกสารได้" |
| Field already has AI-extracted value from another document | Later extraction overwrites (same priority level). Warning shown with previous value. |
| CO edits a pre-filled field then uploads new document | `manual_override` has higher priority → AI extraction does not overwrite |
| Transform function fails | Value not written. Field left blank. Warning logged. |
| Multiple fields share the same `field_path` (shouldn't happen, but defensive) | First write wins within a single extraction report |

---

## Dependencies

| Dependency | Type | Notes |
|-----------|------|-------|
| Document Upload with AI Extraction | Internal — [FEATURE](FEATURE_document-upload-ai-extraction.md) | Provides the `DocumentExtractionReport` |
| Extraction Field Mapping Registry | Internal — [FEATURE](FEATURE_extraction-field-mapping-registry.md) | Provides `check_name → field_path` + threshold + transform |
| Smart Form | Internal — [CAPABILITY](../../smart-form/CAPABILITY.md) | Target for field writes. Owns field definitions and persistence. |
| Field Lockpoint Enforcement | Internal — [FEATURE](../../smart-form/features/FEATURE_field-lockpoint-enforcement.md) | Pre-fill must respect HWM locks |
