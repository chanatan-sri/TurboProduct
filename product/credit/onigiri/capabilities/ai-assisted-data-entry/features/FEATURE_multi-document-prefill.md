# Feature: Multi-Document Pre-fill Orchestration

**Parent Capability**: AI-Assisted Data Entry — [CAPABILITY](../CAPABILITY.md)
**Product**: Onigiri — [PRODUCT](../../../PRODUCT.md)
**Engineering Owner**: TBD
**Status**: Concept
**Last Updated**: 2026-03-30

---

## User Story

As a **Credit Officer**, I want to upload documents of different types and have each document's extracted data fill the appropriate Smart Form sections, so that I can pre-fill the entire form through document uploads without manually cross-referencing which field belongs where.

## Job-to-be-Done

A typical loan application involves multiple document types targeting different Smart Form sections. This feature orchestrates the routing:

| Document Type | Target Smart Form Section |
|--------------|--------------------------|
| Thai National ID Card | Identity, Address |
| Foreigner Passport | Identity (foreigner variant) |
| Vehicle Registration Book (car) | Collateral (car variant) |
| Vehicle Registration Book (bike) | Collateral (bike variant) |
| Vehicle Registration Book (tractor) | Collateral (tractor variant) |
| Land Title Deed | Collateral (land variant) |
| Income Certificate | Occupation, Income & Expenses |
| Bank Statement | Income & Expenses |

The routing is entirely driven by the `target_section` field in the Extraction Field Mapping Registry — no hardcoded routing logic.

---

## Acceptance Criteria

| # | Criterion | Pass Condition |
|---|-----------|---------------|
| AC-1 | Section routing | Each document's extracted fields are routed to the correct Smart Form section based on `target_section` in the mapping registry |
| AC-2 | Independent processing | Documents can be uploaded in any order — each triggers independent extraction and pre-fill |
| AC-3 | Cross-section fill | A single document can fill fields across multiple sections (e.g., ID card fills Identity + Address) |
| AC-4 | Progress indicator | Section headers show which sections have been pre-filled and which still need data |
| AC-5 | Parallel processing | If CO uploads multiple documents simultaneously, extractions run in parallel |
| AC-6 | No duplicate processing | Same image uploaded to the same upload box does not trigger duplicate extraction |
| AC-7 | Section-aware conflict | When two documents target the same field in the same section, Conflict Resolution rules apply |

---

## Multi-Document Flow Example: Bike Title Loan

```
Step 1: DipChip → Identity section filled (source: dipchip)
Step 2: Upload ID card image → Address section supplemented (fields DipChip didn't cover)
Step 3: Upload bike registration book → Collateral (bike) section filled
Step 4: Upload income certificate → Occupation + Income sections filled
Step 5: CO reviews all sections → edits as needed → saves draft
```

**Result:** 4 documents uploaded → 5 Smart Form sections pre-filled → CO only needs to review and correct, not type from scratch.

---

## Progress Indicator

| Section | Status | Indicator |
|---------|--------|-----------|
| Identity | Filled by DipChip + AI | 🔵 "DipChip + AI" |
| Address | Partially filled by AI | 🟡 "AI (3/13 fields — review needed)" |
| Occupation | Filled by AI | 🟢 "AI (8/10 fields)" |
| Income & Expenses | Not yet filled | ⚪ "ยังไม่มีข้อมูล" |
| Collateral (bike) | Filled by AI | 🟢 "AI (12/14 fields)" |
| References | Not applicable for AI | ⚪ "กรอกข้อมูลเอง" |

---

## Edge Cases & Error States

| Scenario | Expected Behavior |
|----------|------------------|
| CO uploads only 1 document | Only that document's target sections are pre-filled; other sections remain for manual entry |
| Document type not configured for current product type | Upload accepted but no extraction triggered (no evidence mapping) |
| CO uploads wrong document type to an upload box | Wasabi type classification catches mismatch → CO prompted to re-upload |
| Two documents of same type uploaded (e.g., two income certs) | Each processed independently; later extraction overwrites earlier for overlapping fields |
| Product type has no collateral section (rare) | Extraction from collateral documents has no target → values ignored |

---

## Dependencies

| Dependency | Type | Notes |
|-----------|------|-------|
| Extraction Field Mapping Registry | Internal — [FEATURE](FEATURE_extraction-field-mapping-registry.md) | `target_section` drives routing logic |
| Smart Form Pre-fill Engine | Internal — [FEATURE](FEATURE_smart-form-prefill-engine.md) | Writes values to resolved fields |
| Pre-fill Conflict Resolution | Internal — [FEATURE](FEATURE_prefill-conflict-resolution.md) | Handles same-field conflicts across documents |
| Document Upload with AI Extraction | Internal — [FEATURE](FEATURE_document-upload-ai-extraction.md) | Triggers extraction per upload |
