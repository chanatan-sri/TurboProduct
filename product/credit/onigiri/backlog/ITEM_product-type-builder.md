# ITEM: Product Type Builder

**Status:** Concept
**Capability:** Product Type Configuration
**Product:** Onigiri
**Owner:** TBD (Engineering — UI)
**Created:** 2026-03-12
**Last Updated:** 2026-03-12

---

## What & Why

Multi-step wizard UI that orchestrates the full Product Owner workflow for assembling a new product type: selecting a collateral section from the engineering-provided registry, registering document types, declaring document requirements with conditional rules, previewing the configuration, and submitting for two-tier approval. This is the integration point between Product Type Configuration and Campaign Configuration — ACTIVE product types become available in the Campaign Builder's Application Template dimension.

## Acceptance Criteria

- [ ] PO can create, save, and resume DRAFT product types across sessions
- [ ] 5-step wizard: Select Section → Register Doc Types → Declare Requirements → Preview → Submit
- [ ] Cross-step validation prevents submission with invalid configuration
- [ ] Preview simulates conditional document rules with sample field values
- [ ] ACTIVE product types appear in Campaign Builder's Application Template picker
- [ ] Campaign pins to product type version at activation (version pinning)

## Dependencies

- Collateral Section Registry (Feature 1) — section picker data
- Document Type Registration (Feature 2) — inline doc type registration
- Document Requirement Declaration (Feature 3) — inline doc requirement config
- Product Type Publication Authorization (Feature 4) — approval workflow + Matcha sync
- Campaign Configuration (external) — consumes ACTIVE product types

## Notes / Open Questions

- Should the wizard enforce a strict step order, or allow PO to jump between steps freely?
- What is the maximum number of document requirements per product type?
- Should there be a "clone from existing product type" shortcut for similar collateral types?

---

## Merge Checklist

*Complete before marking Live and archiving this file.*

- [ ] `FEATURE_product-type-builder.md` created or updated in `capabilities/product-type-configuration/features/`
- [ ] `CAPABILITY.md` feature inventory updated
- [ ] `PRODUCT.md` capability registry updated (if new capability)
- [ ] `BACKLOG.md` row moved to ✅ LIVE
- [ ] `CHANGELOG` entry written
- [ ] This file moved to `backlog/archived/`
