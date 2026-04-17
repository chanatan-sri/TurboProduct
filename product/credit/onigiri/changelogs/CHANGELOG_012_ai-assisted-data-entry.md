# CHANGELOG 012: AI-Assisted Data Entry Capability

**Date**: 2026-03-30
**Layer**: Capability (Onigiri) + Capability Extension (Wasabi)
**Session Type**: @CAPABILITY

---

## What Changed

### New Capability: AI-Assisted Data Entry (Onigiri)

Added a new capability to Onigiri that orchestrates AI-powered document image extraction to pre-fill Smart Form fields during data entry. This inverts the current workflow: instead of CO typing data manually then uploading documents for verification, the CO uploads documents first and AI extracts field values to pre-fill the form.

**7 features decomposed:**
1. **DipChip Integration Gate** — Mandatory chip read before upload boxes appear
2. **Document Upload with AI Extraction Trigger** — Upload → Wasabi extraction mode → quality errors or pre-fill
3. **Extraction Field Mapping Registry** — Extend existing extraction_template with reverse mapping (check_name → field_path)
4. **Smart Form Pre-fill Engine** — Write extracted values to Smart Form with source attribution
5. **Pre-fill Conflict Resolution** — Source priority: DipChip > manual > AI > DaVinci
6. **Extraction Confidence Display** — Green/yellow/gray indicators based on per-field confidence thresholds
7. **Multi-Document Pre-fill Orchestration** — Route fields from different document types to correct sections

### Wasabi Extraction Mode (Operations)

Extended Wasabi's 4-stage pipeline with a new `operational_mode: "extraction"` parameter:
- **Stage 1 (Quality Assessment)**: Unchanged — reused directly
- **Stage 2 (Type Classification)**: Unchanged — reused directly
- **Stage 3 (Instruction Verification)**: New dual-mode — extracts fields without comparison in extraction mode
- **Stage 4 (Report Assembly)**: New `DocumentExtractionReport` format with extracted_fields[] + confidence per field

New extraction-only result statuses: `extracted`, `low_confidence`, `not_extractable` (no `match`/`mismatch` — nothing to compare against).

---

## Rationale

- **Business need**: COs have physical documents in hand before typing. Extracting data from images reduces manual entry time and transcription errors.
- **Architecture**: Wasabi already has LLM extraction (Stage 3). Adding extraction-only mode reuses the existing pipeline without duplication.
- **Template reuse**: Existing `extraction_template` table is bidirectional by nature. Adding inbound mapping fields avoids a parallel registry.
- **Three-way contract preserved**: Matcha config unchanged, PO evidence config unchanged, Onigiri templates extended.

---

## Trade-offs & Rejected Alternatives

| Considered | Rejected Because |
|-----------|-----------------|
| New standalone extraction service | Duplicates Wasabi's quality/classification/extraction stages |
| Build extraction directly in Onigiri | Duplicates LLM infrastructure; violates Wasabi's ownership of document AI |
| New Wasabi endpoint (vs. mode parameter) | Duplicates pipeline orchestration; single endpoint with mode is cleaner |
| Global confidence threshold | Different fields need different thresholds (ID numbers vs. addresses) |
| AI can overwrite DipChip values | DipChip is government chip — authoritative for identity. AI from printed card may have OCR errors. |

---

## Documents Changed

| Document | Change |
|----------|--------|
| `product/credit/onigiri/capabilities/ai-assisted-data-entry/CAPABILITY.md` | **Created** — Full capability spec with 7 features, business rules, error taxonomy, sequence diagrams |
| `product/credit/onigiri/capabilities/ai-assisted-data-entry/features/FEATURE_dipchip-integration-gate.md` | **Created** |
| `product/credit/onigiri/capabilities/ai-assisted-data-entry/features/FEATURE_document-upload-ai-extraction.md` | **Created** |
| `product/credit/onigiri/capabilities/ai-assisted-data-entry/features/FEATURE_extraction-field-mapping-registry.md` | **Created** |
| `product/credit/onigiri/capabilities/ai-assisted-data-entry/features/FEATURE_smart-form-prefill-engine.md` | **Created** |
| `product/credit/onigiri/capabilities/ai-assisted-data-entry/features/FEATURE_prefill-conflict-resolution.md` | **Created** |
| `product/credit/onigiri/capabilities/ai-assisted-data-entry/features/FEATURE_extraction-confidence-display.md` | **Created** |
| `product/credit/onigiri/capabilities/ai-assisted-data-entry/features/FEATURE_multi-document-prefill.md` | **Created** |
| `product/credit/onigiri/backlog/ITEM_dipchip-integration-gate.md` | **Created** |
| `product/credit/onigiri/backlog/ITEM_document-upload-ai-extraction.md` | **Created** |
| `product/credit/onigiri/backlog/ITEM_extraction-field-mapping-registry.md` | **Created** |
| `product/credit/onigiri/backlog/ITEM_smart-form-prefill-engine.md` | **Created** |
| `product/credit/onigiri/backlog/ITEM_prefill-conflict-resolution.md` | **Created** |
| `product/credit/onigiri/backlog/ITEM_extraction-confidence-display.md` | **Created** |
| `product/credit/onigiri/backlog/ITEM_multi-document-prefill.md` | **Created** |
| `product/credit/onigiri/PRODUCT.md` | **Updated** — Added AI-Assisted Data Entry to capability registry; updated Wasabi integration (extraction mode) |
| `product/credit/onigiri/BACKLOG.md` | **Updated** — Added 7 concept items; updated session log |
| `product/operations/wasabi/PRODUCT.md` | **Updated** — Dual-mode value proposition, boundary, capabilities, receives/sends |
| `product/operations/wasabi/capabilities/instruction-verification/CAPABILITY.md` | **Updated** — Dual-mode behavior, extraction statuses, request contract, pre-fill implications |
| `product/operations/wasabi/capabilities/report-assembly/CAPABILITY.md` | **Updated** — DocumentExtractionReport schema, extraction summary counts, extraction human review rules |
