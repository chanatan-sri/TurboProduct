# Feature: Application Template Assignment

**Parent Capability**: Loan Campaign Configuration — [CAPABILITY](../CAPABILITY.md)
**Product**: Onigiri — [PRODUCT](../../../PRODUCT.md)
**Engineering Owner**: TBD
**Status**: Concept
**Last Updated**: 2026-03-18

---

## User Story

As a **Product Manager**, I want to select an ACTIVE product type when building a campaign, so that the campaign inherits the correct application form (sections + variants) and document checklist without configuring them independently.

## Job-to-be-Done

Today, the Campaign Builder's "Application Template" dimension is described as "select which form pages/sections/fields appear" — implying the PM configures form structure directly. In reality, form structure is owned by Product Type Configuration. This feature formalizes the integration: PM selects an ACTIVE product type, and the campaign inherits sections, variants, and document requirements as read-only configuration.

---

## Scope

**IN scope:**
- Browse and select from ACTIVE product types in the Campaign Builder
- Preview the selected product type's sections, variants, and document checklist (read-only)
- Pin the selected product type version at campaign activation
- Validate that a product type is selected before campaign submission

**OUT of scope:**
- Overriding sections, document requirements, or field definitions from the campaign context — these are owned by Product Type
- Creating or editing product types from the Campaign Builder
- Auto-deriving eligibility rules from the product type (collateral type and application type remain independent eligibility criteria configured by PM)

---

## Product Type Selection UI

### Selection Flow

1. PM opens the "Application Template" step in Campaign Builder
2. System displays a list of ACTIVE product types with summary info
3. PM selects one product type
4. System displays a read-only preview showing:
   - Included sections and their selected variants (e.g., Identity: Thai National, Collateral: Bike)
   - Total field count
   - Document checklist (shared base + type-specific, with conditional rules shown)
5. PM confirms selection

### Product Type Browser

| Column | Description |
|--------|-------------|
| Product Type Name | e.g., "Bike Title" |
| Codename | e.g., `bike_title` |
| Collateral Section | e.g., `collateral_bike` |
| Sections | Count of included sections (e.g., 4) |
| Documents | Count of required documents (e.g., 6) |
| Version | Current ACTIVE version number |
| Status | Always ACTIVE (only ACTIVE types shown) |

### Read-Only Preview

After selection, PM sees a summary but **cannot modify** any of the following:

| Preview Section | Content |
|----------------|---------|
| Sections & Variants | Table of included sections, selected variant per section, field count |
| Document Checklist | List of required documents with conditional rules displayed |
| Data Extraction | Template keys assigned per document (informational) |

---

## Integration Rules

### Product Type Version Pinning

When a campaign transitions to ACTIVE, the system captures:

| Field | Value | Mutability |
|-------|-------|-----------|
| `product_type_id` | FK to selected product type | Immutable after activation |
| `product_type_version` | Version number at activation time | Immutable after activation |

**Rules:**
- In-flight applications use the pinned product type version — no retroactive migration
- If PO publishes product type v2, existing ACTIVE campaigns continue with v1
- PM can create a new campaign version selecting v2
- Campaign version + product type version together define the complete application template

### Relationship to Eligibility

Product type selection does **not** auto-derive eligibility rules. The following remain independent PM-configured eligibility criteria:

| Eligibility Criterion | Configured By | Relationship to Product Type |
|-----------------------|--------------|------------------------------|
| `collateral_type` | PM (in Eligibility Rules) | PM ensures alignment manually; no system enforcement |
| `application_type` | PM (in Eligibility Rules) | Independent — gates by transaction nature (new_booking / topup / restructure) |

This is intentional: a single product type may serve multiple campaigns with different eligibility gates.

### Product Type Availability

| Product Type State | Available for Selection? |
|-------------------|------------------------|
| DRAFT | No |
| PENDING_T1 / PENDING_T2 | No |
| ACTIVE | Yes |
| ARCHIVED | No — existing campaigns using it are unaffected |

---

## Acceptance Criteria

| # | Criterion | Pass Condition |
|---|-----------|---------------|
| AC-1 | PM can browse ACTIVE product types | Product type browser shows all ACTIVE types with summary info |
| AC-2 | PM selects a product type | Selection auto-populates read-only preview of sections + documents |
| AC-3 | Version pinning at activation | Campaign activation stores `product_type_id` + `product_type_version`; values immutable after |
| AC-4 | Archived product type handling | If product type is archived while campaign is ACTIVE, campaign continues using pinned version |
| AC-5 | Product type required | PM cannot submit campaign for approval without selecting a product type |

---

## Edge Cases & Error States

| Scenario | Expected Behavior |
|----------|------------------|
| Product type v2 published after campaign activated with v1 | Campaign continues with v1; PM can create new campaign version selecting v2 |
| Product type archived | Existing campaigns unaffected; new campaigns cannot select it |
| Product type has no ACTIVE version | Not shown in browser; PM cannot select it |
| PM deselects product type after selection | Application Template section cleared; campaign cannot be submitted until a new selection is made |
| Campaign versioning | New campaign version can select a different product type (or different version of the same type) |

---

## Dependencies

| Dependency | Type | Notes |
|-----------|------|-------|
| Product Type Configuration | Internal — [CAPABILITY](../../product-type-configuration/CAPABILITY.md) | ACTIVE product types must exist before PM can select them |
| Product Type Builder | Internal — [FEATURE](../../product-type-configuration/features/FEATURE_product-type-builder.md) | PO must have assembled and activated a product type |
| Campaign Builder | Internal | This feature is one step within the Campaign Builder workflow |
