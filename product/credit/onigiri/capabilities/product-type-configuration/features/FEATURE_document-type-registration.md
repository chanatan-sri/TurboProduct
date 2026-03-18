# Feature: Document Type Registration

**Parent Capability**: Product Type Configuration — [CAPABILITY](../CAPABILITY.md)
**Product**: Onigiri — [PRODUCT](../../../PRODUCT.md)
**Engineering Owner**: TBD
**Status**: Concept
**Last Updated**: 2026-03-18

---

## User Story

As an **Engineer**, I want to register document types in Onigiri's registry, so that the AI classification system can categorize uploaded documents and POs can reference these types when declaring evidence requirements.

## Job-to-be-Done

Document types are **AI classification categories** — they tell the document verification system (Matcha) what kind of document it is looking at. Examples: `copy_of_id_card`, `vehicle_registration_book`, `income_certificate`. The same document type can appear in multiple evidence entries (e.g., `copy_of_id_card` is used for both the customer's ID card evidence and the guarantor's ID card evidence — same AI classification, different upload contexts).

Today, Matcha's `DocumentType` table contains ~35 seeded types, added via code migrations. This feature creates an Onigiri-owned registry where Engineering registers document types. At product type activation, Onigiri syncs new types to Matcha via API.

**Why Engineering-owned:** Document types define AI classification boundaries. Creating a new document type requires understanding what the AI model can distinguish and how Matcha's verification instructions are structured. This is a technical decision, not a business configuration.

---

## 3-Level Document Hierarchy

Document Type Registration is the **bottom layer** (AI classification) in Onigiri's 3-level document model:

| Level | What It Is | Owner | Example |
|-------|-----------|-------|---------|
| **Document Type** | AI classification category — tells Matcha what kind of document it's looking at | **Engineering** | `copy_of_id_card`, `vehicle_registration_book` |
| **Evidence** | What the customer must bring to CO — a named requirement configured per section in a product type | **PO** (in Product Type Builder) | "สำเนาบัตรประชาชนผู้กู้" (Customer ID Card), "สำเนาบัตรประชาชนผู้ค้ำ" (Guarantor ID Card) |
| **Upload Box** | A specific upload slot within an evidence — what the CO actually uploads | **PO** (per evidence) | "หน้าปก", "หน้ากรรมสิทธิ์ล่าสุด", "หน้าภาษี" |

**Key insight:** The same document type can map to multiple evidence entries. Both "Customer ID Card" and "Guarantor ID Card" use document type `copy_of_id_card` — same AI classification, different evidence contexts.

---

## Scope

**IN scope:**
- Register new document type keys in Onigiri's `document_type_registry` table
- Define: key (unique identifier), display name (EN + TH), category
- View all registered document types (both Onigiri-registered and pre-existing Matcha-seeded)
- Sync newly registered types to Matcha via `POST /document-types` at product type activation
- Mark types as synced/unsynced with Matcha

**OUT of scope:**
- Matcha verification rule configuration per document type (Matcha owns this)
- Modifying or deleting pre-existing Matcha-seeded document types
- Evidence declaration (PO-owned — see [Document Requirement Declaration](FEATURE_document-requirement-declaration.md))
- Upload box configuration (PO-owned — configured per evidence)

---

## Document Type Registry Schema

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `key` | String | Yes | Unique identifier. Convention: `<descriptive_name>` in snake_case (e.g., `copy_of_id_card`, `vehicle_registration_book`) |
| `display_name` | String | Yes | Human-readable name (e.g., "Copy of ID Card") |
| `display_name_th` | String | No | Thai display name (e.g., "สำเนาบัตรประชาชน") |
| `category` | Enum | Yes | `vehicle`, `land`, `identity`, `income`, `contract`, `insurance`, `other` |
| `source` | Enum | Auto | `onigiri` (registered by Engineering) or `matcha_seed` (pre-existing) |
| `matcha_synced` | Boolean | Auto | `true` once synced to Matcha; `false` until activation |
| `created_by` | String | Auto | Engineer user ID |
| `created_at` | Timestamp | Auto | Registration timestamp |

### Key Naming Convention

System enforces: snake_case, descriptive.

| Valid | Invalid | Why |
|-------|---------|-----|
| `copy_of_id_card` | `CopyOfIDCard` | Not snake_case |
| `vehicle_registration_book` | `document_1` | Not descriptive |
| `income_certificate` | `copy_of_id_card` (duplicate) | Already exists |

---

## Matcha Sync Protocol

```mermaid
sequenceDiagram
    participant ENG as Engineering
    participant Onigiri as Onigiri Admin
    participant Registry as document_type_registry
    participant Matcha as Matcha API

    ENG->>Onigiri: Register new document type
    Onigiri->>Registry: INSERT (matcha_synced = false)
    Note over Registry: Type exists in Onigiri only

    Note over Onigiri: PO uses type in evidence declaration...
    Note over Onigiri: Product type submitted → 2-tier approval

    Onigiri->>Onigiri: CRO approves → activation begins
    Onigiri->>Registry: SELECT WHERE matcha_synced = false AND referenced by product_type

    loop For each unsynced type
        Onigiri->>Matcha: POST /document-types { key, display_name, category }
        alt Success
            Matcha-->>Onigiri: 201 Created
            Onigiri->>Registry: UPDATE matcha_synced = true
        else Failure
            Matcha-->>Onigiri: 4xx/5xx Error
            Onigiri->>Onigiri: Activation blocked; error shown to PO
        end
    end
```

### Failure Handling

| Scenario | Behavior |
|----------|----------|
| Matcha returns 409 (type already exists) | Treat as success — type was previously synced (idempotent) |
| Matcha returns 400 (invalid key format) | Activation blocked; Engineering must fix the key |
| Matcha returns 500 / timeout | Activation blocked; retry available; PO sees "Matcha sync failed" error |
| Partial sync (some types synced, some failed) | Rollback: unmark synced types; activation blocked; retry full sync |

---

## Acceptance Criteria

| # | Criterion | Pass Condition |
|---|-----------|---------------|
| AC-1 | Engineering registers a new document type | Type appears in registry with `matcha_synced = false` |
| AC-2 | System enforces key naming convention | Invalid keys rejected with helpful error message |
| AC-3 | Duplicate key rejected | System prevents registration of a key that already exists (in Onigiri or Matcha seed) |
| AC-4 | Engineering views all document types | Registry shows both Onigiri-registered and Matcha-seeded types with source indicator |
| AC-5 | Activation syncs to Matcha | New types synced via `POST /document-types`; `matcha_synced` updated to `true` |
| AC-6 | Matcha 409 treated as success | Idempotent handling; activation proceeds |
| AC-7 | Matcha failure blocks activation | Product type stays in approval state; PO sees clear error |
| AC-8 | Pre-existing Matcha types are read-only | Cannot edit or delete types with `source = matcha_seed` |
| AC-9 | Registered type available for evidence declaration | New types appear in the evidence configuration's document type picker |

---

## Edge Cases & Error States

| Scenario | Expected Behavior |
|----------|------------------|
| Attempt to delete a type referenced by an evidence entry | Deletion blocked: "Type is referenced by evidence in product type [X]" |
| Matcha API is down during activation | Activation blocked; PO retries later; no partial state |
| Type registered but never referenced by any product type | Type remains in registry with `matcha_synced = false`; no Matcha impact |
| Two engineers register the same key concurrently | Database unique constraint prevents duplicate; second registration sees conflict error |
| Matcha changes its API contract | Engineering updates the sync adapter; no PO-facing changes |
| Same document type used by multiple evidence entries | Normal — document type is a classification, not 1:1 with evidence |

---

## Cross-Product Dependency

**Matcha must expose a document type registration API.** This is a prerequisite for this feature.

| Endpoint | Method | Payload | Response |
|----------|--------|---------|----------|
| `/document-types` | POST | `{ key: string, display_name: string, category: string }` | 201 Created / 409 Conflict / 400 Bad Request |

**Action required:** Create a backlog item in Matcha product for exposing this endpoint.

---

## Dependencies

| Dependency | Type | Notes |
|-----------|------|-------|
| Matcha Document Type Registration API | External — Matcha | Must expose `POST /document-types`; cross-product dependency |
| Product Type Publication Authorization | Internal — [FEATURE](FEATURE_product-type-publication-authorization.md) | Sync happens at activation (after CRO approval) |
| Document Requirement Declaration | Internal — [FEATURE](FEATURE_document-requirement-declaration.md) | Registered types are referenced when PO configures evidence entries |
