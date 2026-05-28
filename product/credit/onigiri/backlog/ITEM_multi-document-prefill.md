# ITEM: Multi-Document Pre-fill Orchestration

**Status:** Concept
**Capability:** AI-Assisted Data Entry
**Product:** Onigiri
**Owner:** TBD
**Created:** 2026-03-30
**Last Updated:** 2026-03-30

---

## What & Why
Route extracted fields from different document types to the correct Smart Form sections. ID card → Identity/Address, vehicle registration → Collateral, income certificate → Occupation/Income. Routing is driven by the `target_section` field in the mapping registry — no hardcoded logic. Enables CO to pre-fill the entire form by uploading multiple documents.

## Acceptance Criteria
- [ ] Each document's fields routed to correct section based on `target_section`
- [ ] Documents uploadable in any order — each triggers independent extraction
- [ ] Single document can fill fields across multiple sections
- [ ] Section headers show pre-fill progress indicator
- [ ] Multiple documents process in parallel when uploaded simultaneously
- [ ] No duplicate extraction for same image in same upload box
- [ ] Cross-document conflicts handled by Conflict Resolution rules

## Dependencies
- Extraction Field Mapping Registry (target_section drives routing)
- Smart Form Pre-fill Engine (writes values)
- Pre-fill Conflict Resolution (handles same-field conflicts)
- Document Upload with AI Extraction (triggers extraction per upload)

## Notes / Open Questions
- Should there be a "batch upload" UX where CO uploads all documents at once?

---

## Merge Checklist
*Complete before marking Live and archiving this file.*
- [ ] `FEATURE_multi-document-prefill.md` created or updated in `capabilities/ai-assisted-data-entry/features/`
- [ ] `CAPABILITY.md` feature inventory updated
- [ ] `PRODUCT.md` capability registry updated (if new capability)
- [ ] `BACKLOG.md` row moved to ✅ LIVE
- [ ] `CHANGELOG` entry written
- [ ] This file moved to `backlog/archived/`
