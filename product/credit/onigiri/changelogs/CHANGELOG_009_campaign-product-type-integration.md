# CHANGELOG 009: Campaign ↔ Product Type Integration + Application Type Eligibility

**Date**: 2026-03-18
**Layer**: @FEATURE (Loan Campaign Configuration) + @CAPABILITY (cross-capability integration)
**Type**: Feature Design + Eligibility Enhancement

---

## What Changed

### 1. Created Application Template Assignment Feature Spec

New feature spec for Campaign Configuration's integration with Product Type:

- PM selects an ACTIVE product type in Campaign Builder → sections + variants + document checklist inherited as read-only
- Product type version pinned at campaign activation (`product_type_id` + `product_type_version` immutable)
- In-flight applications always use the pinned version — no retroactive migration
- PM cannot override sections, documents, or field definitions — these are owned by Product Type

### 2. Added Application Type Eligibility Field

New **required** eligibility criterion classifying the loan transaction nature:

| Application Type | Key | Description |
|-----------------|-----|-------------|
| New Booking | `new_booking` | New loan with new collateral |
| Topup | `topup` | Additional loan on existing collateral (retention) |
| Restructure | `restructure` | Restructuring of an existing loan |

Added to the Eligibility Criteria Examples table in Campaign CAPABILITY.md.

### 3. Corrected Collateral Type Independence

**Previous assumption (incorrect):** Collateral type would be auto-derived from the selected product type's collateral section, with system-enforced coupling.

**Correction:** Collateral type remains an **independent** eligibility criterion configured by the PM. No system auto-coupling between the product type's collateral section and the campaign's `collateral_type` eligibility rule. This is intentional — a single product type may serve multiple campaigns with different eligibility gates.

### 4. Updated Stale Campaign Configuration Text

- "Application Template" dimension: replaced "select which form pages/sections/fields appear" → "select ACTIVE Product Type — auto-populates sections + documents"
- "Application Template Assignment" feature: updated description + linked to FEATURE spec
- "How document list reaches Matcha" paragraph: clarified documents come from product type, not campaign
- User flow diagram: updated Application Template step

---

## Rationale

The Campaign Builder's "Application Template" dimension was described as if the PM configures form structure directly. In reality, form structure is owned by Product Type Configuration. This change formalizes the integration boundary:

- **Product Type** owns: sections, variants, document requirements, extraction templates
- **Campaign** owns: pricing, eligibility, risk strategy, workflow steps + product type reference

Application Type was added because campaigns need to differentiate between new bookings, topups, and restructures at the eligibility gate. This affects which campaign a customer enters and may also affect which product type is appropriate (design deferred).

---

## Decision Log

| Decision | Choice | Alternatives Considered | Why |
|----------|--------|------------------------|-----|
| Collateral type coupling | Independent (no auto-coupling) | Auto-derive from product type; system validation | PM needs flexibility — a single product type may serve campaigns with different eligibility gates. Auto-coupling would be too rigid. |
| Application Type as eligibility field | Required field with 3 values | Optional field; separate dimension; embedded in product type | It's a gate that determines which campaign applies — belongs in eligibility. Required because every campaign must classify its transaction type. |
| Application Type ↔ Product Type interaction | Deferred to future session | Design now | User acknowledged the interaction exists but doesn't have a clear design yet. Better to document the open question than over-spec. |
| Product type version pinning | Immutable after campaign activation | Allow campaign to auto-upgrade; manual upgrade with migration | Immutability prevents retroactive changes to in-flight applications. PM creates new campaign version to adopt product type v2. |

---

## Documents Changed

| Document | Change |
|----------|--------|
| `loan-campaign-configuration/features/FEATURE_application-template-assignment.md` | **New** — integration spec: PM selects ACTIVE product type |
| `loan-campaign-configuration/CAPABILITY.md` | Updated stale text, added version pinning + Application Type sections, updated eligibility table + document flow + user flow diagram |
| `product-type-configuration/CAPABILITY.md` | Added campaign integration cross-reference + version pinning to Mermaid diagram |
| `backlog/ITEM_application-template-assignment.md` | **New** — backlog item card |
| `BACKLOG.md` | Added item to CONCEPT + session log entry |
