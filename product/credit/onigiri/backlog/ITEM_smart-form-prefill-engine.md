# ITEM: Smart Form Pre-fill Engine

**Status:** Concept
**Capability:** AI-Assisted Data Entry
**Product:** Onigiri
**Owner:** TBD
**Created:** 2026-03-30
**Last Updated:** 2026-03-30

---

## What & Why
Core runtime engine that receives extracted fields from Wasabi, resolves target Smart Form fields via the mapping registry, applies confidence thresholds, and writes values with source attribution metadata. This is what makes the pre-fill happen — transforming raw extraction results into populated form fields.

## Acceptance Criteria
- [ ] Each extracted check_name resolved to Smart Form field_path via mapping registry
- [ ] Fields filled based on confidence: >= threshold (auto), >= 0.70 (warning), < 0.70 (blank)
- [ ] Every pre-filled field tagged with `source: "ai_extraction"`, document_id, confidence
- [ ] Pre-filled fields visually distinct from manual fields
- [ ] All pre-filled values editable by CO
- [ ] Cross-section fill works (fields in unvisited sections pre-filled)
- [ ] Values persisted to DocumentDB on save-draft
- [ ] Locked fields (HWM) not overwritten

## Dependencies
- Document Upload with AI Extraction (provides extraction report)
- Extraction Field Mapping Registry (provides field mappings)
- Smart Form (target for writes)
- Field Lockpoint Enforcement (respects HWM locks)

## Notes / Open Questions
- Should pre-fill metadata be stored separately for audit trail?

---

## Merge Checklist
*Complete before marking Live and archiving this file.*
- [ ] `FEATURE_smart-form-prefill-engine.md` created or updated in `capabilities/ai-assisted-data-entry/features/`
- [ ] `CAPABILITY.md` feature inventory updated
- [ ] `PRODUCT.md` capability registry updated (if new capability)
- [ ] `BACKLOG.md` row moved to ✅ LIVE
- [ ] `CHANGELOG` entry written
- [ ] This file moved to `backlog/archived/`
