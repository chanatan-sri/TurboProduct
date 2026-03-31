# ITEM: External Insurance Reference Lookup

**Status:** Concept
**Capability:** Insurance Integration
**Product:** Onigiri
**Owner:** TBD
**Created:** 2026-03-30
**Last Updated:** 2026-03-30

---

## What & Why
Allow the CO to enter a reference number from a customer's externally purchased voluntary/compulsory insurance policy. The system calls the external insurance API to validate the reference and retrieve policy details (premium, coverage, expiry, status). Multiple references can be added per application. This integrates external insurance purchases into the loan calculation without manual premium entry.

## Acceptance Criteria
- [ ] CO enters reference number and clicks "Look up"
- [ ] System calls `GET /insurance/reference/{ref_number}` and displays policy details
- [ ] Active policies can be confirmed and added; expired/cancelled policies are rejected with clear error
- [ ] CO can add multiple references (multi-reference support)
- [ ] CO can remove confirmed references before lockpoint
- [ ] Duplicate reference detection prevents same reference being added twice
- [ ] Each add/remove triggers Ontop/Deduct recalculation

## Dependencies
- External Insurance System API
- Insurance Section in Smart Form
- Insurance Premium Ontop/Deduct Calculator
- Product Type Configuration (`voluntary_insurance` flag)

## Notes / Open Questions
- Is there a maximum number of voluntary/compulsory references per application?
- Reference number format validation rules TBD

---

## Merge Checklist
*Complete before marking Live and archiving this file.*
- [ ] `FEATURE_external-insurance-reference-lookup.md` created or updated in `capabilities/insurance-integration/features/`
- [ ] `CAPABILITY.md` feature inventory updated
- [ ] `PRODUCT.md` capability registry updated (if new capability)
- [ ] `BACKLOG.md` row moved to ✅ LIVE
- [ ] `CHANGELOG` entry written
- [ ] This file moved to `backlog/archived/`
