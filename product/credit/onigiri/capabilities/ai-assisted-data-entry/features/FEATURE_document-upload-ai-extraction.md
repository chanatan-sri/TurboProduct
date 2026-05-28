# Feature: Document Upload with AI Extraction Trigger

**Parent Capability**: AI-Assisted Data Entry — [CAPABILITY](../CAPABILITY.md)
**Product**: Onigiri — [PRODUCT](../../../PRODUCT.md)
**Engineering Owner**: TBD
**Status**: Concept
**Last Updated**: 2026-03-30

---

## User Story

As a **Credit Officer**, I want to upload a document image and have the system automatically send it for AI processing, so that I receive immediate feedback on document quality and extracted data without manual effort.

## Job-to-be-Done

When a CO uploads a document image (after DipChip success), this feature:
1. Sends the image to Wasabi in extraction mode (`operational_mode: "extraction"`)
2. Shows a loading state while Wasabi processes
3. On quality failure → displays the specific error (blur, truncated, etc.) with Thai-language guidance
4. On type mismatch → displays a warning with the detected type
5. On success → passes extracted fields to the Smart Form Pre-fill Engine

This is the bridge between the upload UI and Wasabi's extraction pipeline.

---

## Acceptance Criteria

| # | Criterion | Pass Condition |
|---|-----------|---------------|
| AC-1 | Auto-trigger on upload | On image upload, Onigiri automatically sends extraction request to Wasabi — no manual "extract" button needed |
| AC-2 | Request payload | Request includes: `fileUrls`, `documentTypeKey` (from evidence config), `operational_mode: "extraction"` |
| AC-3 | Loading state | Upload box shows processing indicator while Wasabi processes (up to 10s) |
| AC-4 | Quality error display | On quality failure: show error code + Thai message + guidance (e.g., "ภาพเบลอ ไม่ชัดเจน — กรุณาถ่ายใหม่") |
| AC-5 | Blocking errors prevent pre-fill | Blocking quality errors (BLUR, DARK_IMAGE, TRUNCATED, OBSTRUCTED, GLARE) → no extraction results; CO must re-upload |
| AC-6 | Type mismatch display | On type mismatch: show expected vs. detected type; CO can re-upload correct document |
| AC-7 | Success handoff | On successful extraction: pass `DocumentExtractionReport.extracted_fields[]` to Pre-fill Engine |
| AC-8 | Re-upload replaces | Re-uploading a document for the same upload box replaces the previous image and triggers new extraction |
| AC-9 | Multiple upload boxes | Each upload box in an evidence group independently triggers extraction |
| AC-10 | Wasabi unavailable | If Wasabi is unreachable or returns error: show warning, allow CO to enter data manually |

---

## Edge Cases & Error States

| Scenario | Expected Behavior |
|----------|------------------|
| Wasabi timeout (>10s) | Show: "การประมวลผลใช้เวลานาน กรุณารอสักครู่" — continue waiting up to 30s; then fallback to manual |
| Wasabi returns PROCESSING_FAILED | Show warning: "ระบบไม่สามารถประมวลผลเอกสารได้ กรุณาลองใหม่หรือกรอกข้อมูลเอง" |
| CO navigates away during processing | Extraction continues in background; results applied when CO returns to the section |
| Same document uploaded to two different upload boxes | Each triggers independent extraction — results may overlap if targeting same fields (handled by Conflict Resolution) |
| Image file too large | Client-side validation: max 10MB per image; show error before upload |
| Unsupported file format | Client-side validation: accept only JPEG, PNG, PDF (single page); reject others |

---

## Dependencies

| Dependency | Type | Notes |
|-----------|------|-------|
| DipChip Integration Gate | Internal — [FEATURE](FEATURE_dipchip-integration-gate.md) | Upload boxes only visible after DipChip success |
| Wasabi Extraction Mode | External — Wasabi | Wasabi must support `operational_mode: "extraction"` |
| Product Type evidence config | Internal — [FEATURE](../../product-type-configuration/features/FEATURE_document-requirement-declaration.md) | Upload boxes are configured per evidence per section |
| Smart Form Pre-fill Engine | Internal — [FEATURE](FEATURE_smart-form-prefill-engine.md) | Receives extraction results for field population |
