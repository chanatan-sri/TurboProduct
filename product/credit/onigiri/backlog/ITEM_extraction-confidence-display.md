# ITEM: Extraction Confidence Display

**Status:** Concept
**Capability:** AI-Assisted Data Entry
**Product:** Onigiri
**Owner:** TBD
**Created:** 2026-03-30
**Last Updated:** 2026-03-30

---

## What & Why
Show visual confidence indicators on AI-extracted fields so Credit Officers know which values to double-check. Green = high confidence, yellow = low confidence (please verify), gray = could not extract. Reduces CO review time by focusing attention on uncertain values.

## Acceptance Criteria
- [ ] Green AI icon for fields >= confidence threshold
- [ ] Yellow AI icon with "please verify" for fields >= 0.70 and < threshold
- [ ] Gray AI icon for fields < 0.70 (left blank)
- [ ] DipChip-sourced fields show distinct chip icon
- [ ] AI indicator clears when CO edits field
- [ ] Tooltip shows confidence % and source document name
- [ ] Per-field thresholds supported (from mapping registry)
- [ ] Section header shows count of AI-filled fields and fields needing review

## Dependencies
- Smart Form Pre-fill Engine (provides source + confidence metadata)
- Extraction Field Mapping Registry (provides per-field thresholds)
- Smart Form UI (renders indicators)

## Notes / Open Questions
- Should confidence indicators persist after save, or only during the current session?

---

## Merge Checklist
*Complete before marking Live and archiving this file.*
- [ ] `FEATURE_extraction-confidence-display.md` created or updated in `capabilities/ai-assisted-data-entry/features/`
- [ ] `CAPABILITY.md` feature inventory updated
- [ ] `PRODUCT.md` capability registry updated (if new capability)
- [ ] `BACKLOG.md` row moved to ✅ LIVE
- [ ] `CHANGELOG` entry written
- [ ] This file moved to `backlog/archived/`
