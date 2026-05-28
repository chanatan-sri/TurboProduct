# Feature: Extraction Field Mapping Registry

**Parent Capability**: AI-Assisted Data Entry — [CAPABILITY](../CAPABILITY.md)
**Product**: Onigiri — [PRODUCT](../../../PRODUCT.md)
**Engineering Owner**: TBD
**Status**: Concept
**Last Updated**: 2026-03-30

---

## User Story

As an **Engineer**, I want to define bidirectional mappings between Wasabi-extractable fields and Smart Form field paths per document type, so that the system knows where to place extracted values in the form and which confidence thresholds to apply.

## Job-to-be-Done

The existing `extraction_template` table already maps `check_name ↔ field_path` for outbound data to Matcha. This feature extends those templates with inbound (extraction → Smart Form) metadata, making the mapping bidirectional:

- **Outbound** (existing): `field_path` → `check_name` → sent to Matcha
- **Inbound** (new): `check_name` → `field_path` → written to Smart Form

No new table is needed — the existing template is extended with new columns.

---

## Schema Extension

**New fields per mapping entry in `extraction_template`:**

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `extraction_enabled` | boolean | `true` | Whether this field should be requested/accepted in extraction mode |
| `target_section` | string | — | Which Smart Form section this field maps to (e.g., `identity`, `collateral_car`, `address`) |
| `confidence_threshold` | number | `0.90` | Minimum confidence for auto-fill. Below threshold but >= 0.70: fill with warning. Below 0.70: leave blank. |
| `value_transform_inbound` | string? | `null` | Optional transformation when writing to Smart Form (e.g., `date_buddhist_to_gregorian`, `split_fullname`) |

---

## Acceptance Criteria

| # | Criterion | Pass Condition |
|---|-----------|---------------|
| AC-1 | Extend existing template table | New columns added to `extraction_template` without breaking existing outbound flow |
| AC-2 | Per-field extraction toggle | `extraction_enabled = false` → field skipped during extraction pre-fill (even if Wasabi returns it) |
| AC-3 | Per-field confidence threshold | Pre-fill Engine uses field-specific threshold, not a global one |
| AC-4 | Target section resolution | `target_section` correctly routes extracted values to the right Smart Form section |
| AC-5 | Value transformation | `value_transform_inbound` applied before writing to Smart Form (e.g., Buddhist year → Gregorian) |
| AC-6 | Variant-specific templates | Same `check_name` maps to different `field_path` for different collateral variants (car/bike/tractor/land) — handled by separate template per variant |
| AC-7 | Template seeded for all document types | Initial templates created for: Thai ID card, foreigner passport, vehicle registration (car, bike, tractor), land title deed, income certificate |
| AC-8 | Backward compatible | Existing outbound flow (Onigiri Worker → Matcha) unchanged by new columns |

---

## Example Templates

### Thai ID Card (`thai_id_card_standard`)

| check_name | field_path | extraction_enabled | target_section | confidence_threshold |
|-----------|-----------|:-:|----------------|:-:|
| `id_card_number` | `identity.id_national_id` | true | `identity` | 0.95 |
| `prefix_th` | `identity.id_prefix` | true | `identity` | 0.90 |
| `first_name_th` | `identity.id_first_name` | true | `identity` | 0.90 |
| `last_name_th` | `identity.id_last_name` | true | `identity` | 0.90 |
| `date_of_birth` | `identity.id_date_of_birth` | true | `identity` | 0.90 |
| `address_text` | `address.addr_current_*` | true | `address` | 0.80 |

### Car Registration Book (`vehicle_registration_book_car`)

| check_name | field_path | extraction_enabled | target_section | confidence_threshold |
|-----------|-----------|:-:|----------------|:-:|
| `plate_number` | `collateral_car.car_plate_number` | true | `collateral_car` | 0.95 |
| `plate_province` | `collateral_car.car_plate_province` | true | `collateral_car` | 0.90 |
| `brand` | `collateral_car.car_brand` | true | `collateral_car` | 0.90 |
| `model` | `collateral_car.car_model` | true | `collateral_car` | 0.90 |
| `year` | `collateral_car.car_year` | true | `collateral_car` | 0.90 |
| `chassis_number` | `collateral_car.car_chassis_number` | true | `collateral_car` | 0.95 |
| `engine_number` | `collateral_car.car_engine_number` | true | `collateral_car` | 0.95 |
| `engine_cc` | `collateral_car.car_engine_cc` | true | `collateral_car` | 0.90 |
| `color` | `collateral_car.car_color` | true | `collateral_car` | 0.85 |
| `vehicle_type` | `collateral_car.car_vehicle_type` | true | `collateral_car` | 0.90 |

### Bike Registration Book (`vehicle_registration_book_bike`)

| check_name | field_path | extraction_enabled | target_section | confidence_threshold |
|-----------|-----------|:-:|----------------|:-:|
| `plate_number` | `collateral_bike.bike_plate_number` | true | `collateral_bike` | 0.95 |
| `plate_province` | `collateral_bike.bike_plate_province` | true | `collateral_bike` | 0.90 |
| `brand` | `collateral_bike.bike_brand` | true | `collateral_bike` | 0.90 |
| `model` | `collateral_bike.bike_model` | true | `collateral_bike` | 0.90 |
| `year` | `collateral_bike.bike_year` | true | `collateral_bike` | 0.90 |
| `chassis_number` | `collateral_bike.bike_chassis_number` | true | `collateral_bike` | 0.95 |
| `engine_number` | `collateral_bike.bike_engine_number` | true | `collateral_bike` | 0.95 |
| `engine_cc` | `collateral_bike.bike_engine_cc` | true | `collateral_bike` | 0.90 |

---

## Edge Cases & Error States

| Scenario | Expected Behavior |
|----------|------------------|
| Wasabi returns a `check_name` not in the template | Ignored — value not written to any Smart Form field. Logged for engineering review. |
| Template references a `field_path` that doesn't exist in the active variant | Value not written. Warning logged. |
| `extraction_enabled = false` for a field but Wasabi returns it | Value ignored — not written to Smart Form |
| `value_transform_inbound` fails (e.g., unparseable date) | Value not written. Field left for manual entry. Warning shown. |
| New document type registered without extraction template | Extraction mode still triggers (quality + type classification work), but no fields pre-filled. Warning to engineering. |

---

## Dependencies

| Dependency | Type | Notes |
|-----------|------|-------|
| Existing `extraction_template` table | Internal | Extended with new columns — not replaced |
| Smart Form Section & Variant Registry | Internal — [CAPABILITY](../../smart-form/CAPABILITY.md) | `field_path` references must match Smart Form field keys |
| Matcha DocumentVerificationItem | External | `check_name` values must match Matcha's configuration |
| Document Type Registration | Internal — [FEATURE](../../product-type-configuration/features/FEATURE_document-type-registration.md) | Template must reference a registered document type |
