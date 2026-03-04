# Capability: Master Data

**Capability Name**: Organisational & Master Data
**Parent Product**: Bookkeeping → [PRODUCT](../../PRODUCT.md)
**Product Owner**: Phasathon & Pojchara
**Status**: 📝 Draft
**Last Updated**: 2026-03-04

---

## Business Function

Manages the organisational reference tables that support all other Bookkeeping capabilities. This includes entity registry, reference field definitions, accounting type catalogue, and sub-ledger type catalogue. Master Data is a foundational, cross-capability shared reference layer — it does not process transactions but its data is required by COA Management, Accounting Gateway, and Accounting Book to function correctly.

---

## Feature Inventory

| ID | Feature | Description | Priority | Status |
|----|---------|-------------|---------|--------|
| — | Entity Registry | Manage NTB legal entities and business units referenced in transactions | P1 | 📝 Spec |
| — | Reference Field Definitions | Define and manage Reference1–Reference6 field metadata and constraints | P1 | 📝 Spec |
| — | Accounting Type Catalogue | Manage accounting type codes referenced by event configurations | P1 | 📝 Spec |
| — | Sub-Ledger Type Catalogue | Manage the valid sub-ledger types (Customer, Vendor, None) | P1 | 📝 Spec |

> Note: Master Data features do not have assigned F-numbers in Release 1. They are managed as seed data and API configuration. Feature IDs will be assigned in a future changelog if self-service management is added.

---

## Key Entities

| Entity | Table | Purpose |
|--------|-------|---------|
| Entity | `master_entity` | NTB legal entities and business units. Referenced in transaction headers. |
| Reference Field | `master_reference_field` | Defines the metadata and validation rules for Reference1–Reference6 fields used across transaction types. |
| Accounting Type | `master_accounting_type` | Catalogue of accounting types. Used by event config to classify transaction categories. |
| Sub-Ledger Type | `master_sub_ledger_type` | Defines valid sub-ledger types: Customer, Vendor, None. Assigned to GL accounts via COA Management (F3). |

---

## Business Rules

| Rule | Detail |
|------|--------|
| Entity uniqueness | Each entity must have a unique identifier. Used for multi-entity accounting segregation. |
| Reference field constraints | Each reference field definition specifies whether it is required or optional per event type. Enforced at Accounting Gateway Stage 2. |
| Sub-ledger type immutability | Sub-ledger type values (Customer, Vendor, None) are a fixed catalogue — they are not user-configurable at runtime. |
| Accounting type scope | Accounting types are used purely for classification and reporting grouping — they do not affect GL resolution logic. |

---

## Non-Functional Requirements

| NFR | Requirement |
|-----|-------------|
| Availability | Master data must be available at all times — blocking dependency for all transaction validation |
| Seed data integrity | All master data tables are pre-seeded at system deployment. Changes require dev team intervention in Release 1. |
| Change audit | All master data changes must be logged with timestamp and actor |

---

## Open Questions & Constraints

| # | Question | Status |
|---|----------|--------|
| 1 | Is there a self-service UI planned for master data management in a future release? | Open |
| 2 | How are reference field requirements communicated to upstream systems (LOS, Cash Reconcile)? | Open |
| 3 | Are entities (master_entity) managed by Bookkeeping or synced from DaVinci (Platform portfolio)? | Open — likely a cross-portfolio dependency to resolve |
