# ITEM: Document Upload with AI Extraction Trigger

**Status:** Concept
**Capability:** AI-Assisted Data Entry
**Product:** Onigiri
**Owner:** TBD
**Created:** 2026-03-30
**Last Updated:** 2026-03-30

---

## What & Why
When a CO uploads a document image (after DipChip), automatically send it to Wasabi in extraction mode and display the results — either quality errors requiring re-upload or extracted fields passed to the pre-fill engine. This is the bridge between the upload UI and Wasabi's extraction pipeline.

## Acceptance Criteria
- [ ] On image upload, Onigiri automatically sends extraction request to Wasabi
- [ ] Request includes `fileUrls`, `documentTypeKey`, `operational_mode: "extraction"`
- [ ] Loading state shown during Wasabi processing
- [ ] Quality errors displayed with Thai-language guidance
- [ ] Blocking errors prevent pre-fill; CO must re-upload
- [ ] Type mismatch shows expected vs. detected type
- [ ] Successful extraction hands off to Pre-fill Engine
- [ ] Wasabi unavailability falls back to manual entry

## Dependencies
- DipChip Integration Gate (upload boxes only visible after DipChip)
- Wasabi extraction mode (operational_mode: "extraction")
- Smart Form Pre-fill Engine (receives extraction results)

## Notes / Open Questions
- Maximum image file size? (Proposed: 10MB)
- Supported formats? (Proposed: JPEG, PNG, single-page PDF)

---

## Merge Checklist
*Complete before marking Live and archiving this file.*
- [ ] `FEATURE_document-upload-ai-extraction.md` created or updated in `capabilities/ai-assisted-data-entry/features/`
- [ ] `CAPABILITY.md` feature inventory updated
- [ ] `PRODUCT.md` capability registry updated (if new capability)
- [ ] `BACKLOG.md` row moved to ✅ LIVE
- [ ] `CHANGELOG` entry written
- [ ] This file moved to `backlog/archived/`
