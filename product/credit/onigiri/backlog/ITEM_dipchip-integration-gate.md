# ITEM: DipChip Integration Gate

**Status:** Concept
**Capability:** AI-Assisted Data Entry
**Product:** Onigiri
**Owner:** TBD
**Created:** 2026-03-30
**Last Updated:** 2026-03-30

---

## What & Why
Require a successful DipChip ID card chip read before showing document upload options during data entry. This ensures baseline identity data is always captured from the authoritative government chip source first, preventing AI extraction from overwriting authoritative identity fields.

## Acceptance Criteria
- [ ] Document upload UI hidden/disabled until DipChip read succeeds
- [ ] DipChip fields (name, ID, DoB, address) written to Smart Form identity section
- [ ] Each DipChip-populated field tagged with `source: "dipchip"`
- [ ] Upload boxes appear after successful chip read
- [ ] DipChip is mandatory — no bypass path
- [ ] Failed chip read shows error with retry option

## Dependencies
- DipChip hardware integration (external)
- Smart Form identity section field definitions

## Notes / Open Questions
- How to handle foreigner identity variant (no Thai ID chip)?
- Should there be a supervisor-approved bypass for branches without DipChip hardware?

---

## Merge Checklist
*Complete before marking Live and archiving this file.*
- [ ] `FEATURE_dipchip-integration-gate.md` created or updated in `capabilities/ai-assisted-data-entry/features/`
- [ ] `CAPABILITY.md` feature inventory updated
- [ ] `PRODUCT.md` capability registry updated (if new capability)
- [ ] `BACKLOG.md` row moved to ✅ LIVE
- [ ] `CHANGELOG` entry written
- [ ] This file moved to `backlog/archived/`
