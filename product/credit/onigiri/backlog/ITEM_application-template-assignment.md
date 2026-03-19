# ITEM: Application Template Assignment

**Status:** Concept
**Capability:** Loan Campaign Configuration
**Product:** Onigiri
**Owner:** TBD
**Created:** 2026-03-18
**Last Updated:** 2026-03-18

---

## What & Why
Campaign Builder feature that allows a Product Manager to select an ACTIVE product type, so that the campaign inherits the correct application form (sections + variants) and document checklist as read-only configuration. Replaces the stale "select which form pages/sections/fields appear" description. Includes product type version pinning at campaign activation — in-flight applications always use the pinned version.

## Acceptance Criteria
- [ ] PM can browse and select from ACTIVE product types in Campaign Builder
- [ ] Selecting a product type shows read-only preview of sections, variants, and document checklist
- [ ] Campaign activation captures `product_type_id` + `product_type_version` (immutable)
- [ ] Archived product types do not break existing campaigns (pinned version continues to work)
- [ ] Campaign cannot be submitted for approval without a product type selected

## Dependencies
- Product Type Configuration capability (ACTIVE product types must exist)
- Product Type Builder feature (PO must have assembled and activated a product type)
- Campaign Builder feature (this is one step within the Campaign Builder workflow)

## Notes / Open Questions
- How does Application Type (new_booking / topup / restructure) interact with Product Type? Does each application type require a different product type with different sections/fields? Design deferred to a future @CAPABILITY session.

---

## Merge Checklist
*Complete before marking Live and archiving this file.*
- [ ] `FEATURE_application-template-assignment.md` created or updated in `capabilities/loan-campaign-configuration/features/`
- [ ] `CAPABILITY.md` feature inventory updated
- [ ] `PRODUCT.md` capability registry updated (if new capability)
- [ ] `BACKLOG.md` row moved to ✅ LIVE
- [ ] `CHANGELOG` entry written
- [ ] This file moved to `backlog/archived/`
