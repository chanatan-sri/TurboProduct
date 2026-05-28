# ITEM: Extraction Field Mapping Registry

**Status:** Concept
**Capability:** AI-Assisted Data Entry
**Product:** Onigiri
**Owner:** TBD
**Created:** 2026-03-30
**Last Updated:** 2026-03-30

---

## What & Why
Extend the existing `extraction_template` table with reverse mapping fields (check_name → Smart Form field_path) plus extraction_enabled, confidence_threshold, and value_transform_inbound columns. This makes existing templates bidirectional — supporting both outbound (to Matcha) and inbound (from Wasabi extraction) flows without creating a parallel registry.

## Acceptance Criteria
- [ ] New columns added to `extraction_template` without breaking existing outbound flow
- [ ] Per-field `extraction_enabled` toggle controls which fields participate in pre-fill
- [ ] Per-field `confidence_threshold` used by Pre-fill Engine
- [ ] `target_section` correctly routes values to the right Smart Form section
- [ ] `value_transform_inbound` applied before writing (e.g., date format conversion)
- [ ] Variant-specific templates (car/bike/tractor/land) route to correct collateral sections
- [ ] Templates seeded for all current document types

## Dependencies
- Existing `extraction_template` table
- Smart Form Section & Variant Registry (field paths)
- Matcha DocumentVerificationItem (check_name values)

## Notes / Open Questions
- Should template seeding be automated from Matcha's DocumentVerificationItem config?
- Need alignment with Matcha team on any new check_name keys

---

## Merge Checklist
*Complete before marking Live and archiving this file.*
- [ ] `FEATURE_extraction-field-mapping-registry.md` created or updated in `capabilities/ai-assisted-data-entry/features/`
- [ ] `CAPABILITY.md` feature inventory updated
- [ ] `PRODUCT.md` capability registry updated (if new capability)
- [ ] `BACKLOG.md` row moved to ✅ LIVE
- [ ] `CHANGELOG` entry written
- [ ] This file moved to `backlog/archived/`
