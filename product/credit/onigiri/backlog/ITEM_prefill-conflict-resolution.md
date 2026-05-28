# ITEM: Pre-fill Conflict Resolution

**Status:** Concept
**Capability:** AI-Assisted Data Entry
**Product:** Onigiri
**Owner:** TBD
**Created:** 2026-03-30
**Last Updated:** 2026-03-30

---

## What & Why
Define and enforce source priority rules when multiple data sources (DipChip, AI extraction, manual entry, DaVinci) provide values for the same Smart Form field. DipChip is authoritative for identity; AI extraction never overwrites higher-priority sources. This prevents incorrect data from silently replacing correct data.

## Acceptance Criteria
- [ ] AI extraction never overwrites DipChip, manual_override, or manual_entry values
- [ ] Source tag updated when higher-priority source overwrites
- [ ] Previous value preserved in metadata when same-priority overwrite occurs
- [ ] CO notified when two AI extractions target the same field
- [ ] CO editing any field sets source to `manual_override`
- [ ] Priority hierarchy: dipchip > manual_override > manual_entry > ai_extraction > prefill_davinci

## Dependencies
- Smart Form Pre-fill Engine (calls Conflict Resolution before writing)
- DipChip Integration Gate (DipChip values must be tagged first)
- Source attribution model in application JSON

## Notes / Open Questions
- Should empty DipChip values be treated as "no value" (allowing AI to fill) or "authoritative empty"?

---

## Merge Checklist
*Complete before marking Live and archiving this file.*
- [ ] `FEATURE_prefill-conflict-resolution.md` created or updated in `capabilities/ai-assisted-data-entry/features/`
- [ ] `CAPABILITY.md` feature inventory updated
- [ ] `PRODUCT.md` capability registry updated (if new capability)
- [ ] `BACKLOG.md` row moved to ✅ LIVE
- [ ] `CHANGELOG` entry written
- [ ] This file moved to `backlog/archived/`
