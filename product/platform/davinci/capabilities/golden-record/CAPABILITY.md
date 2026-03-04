# Capability: Golden Record

**Product**: DaVinci — [PRODUCT](../../PRODUCT.md)
**Portfolio**: Platform
**Product Owner**: TBD (Platform PO)
**Status**: 📝 Draft — @FEATURE decomposition pending
**Last Updated**: 2026-03-04

---

## Business Function

Provide a single, authoritative, deduplicated customer profile — the "Golden Record" — that aggregates identity and product summary data from all source systems and serves as the canonical customer reference for all downstream consumers. Every system that needs to know "who is this customer?" or "what products do they hold?" queries DaVinci.

## Why It Exists (First Principles)

- **Regulatory Risk**: Thai Debt Collection Act limits contact frequency per customer per day. Without a combined view, separate teams may unknowingly exceed the limit by contacting the same customer for different products.
- **Operational Friction**: Address changes, phone number updates, and KYC re-verifications must be repeated across every system without a central record.
- **Missed Opportunity**: Cross-sell and upsell opportunities are invisible when customer data is siloed by product.

---

## Feature Inventory

| Feature | Status | Description |
|---------|--------|-------------|
| Customer Identity Store | Draft | Name (TH/EN), DOB, National ID (encrypted), Passport, Tax ID per customer |
| Unique Customer ID Generator | Draft | System-generated `davinci_customer_id` as canonical key across all downstream systems |
| Contact Profile Manager | Draft | Phone numbers (ranked primary/secondary), Email, Physical addresses (with type: home/work/mailing) |
| KYC/AML Status Tracker | Draft | Current verification status, last verification date, source system of verification |
| Product Summary Linkage | Draft | Loan and insurance product summaries linked to customer (not full records — summaries only) |
| Multi-Subsidiary Awareness | Draft | Every customer record and product linkage carries `originating_subsidiary_id` |

---

## Business Rules

### Identity Core Fields

| Field | Type | Notes |
|-------|------|-------|
| `davinci_customer_id` | UUID (system-generated) | Canonical key; immutable once created |
| `name_th` | String | Thai name |
| `name_en` | String | English name |
| `date_of_birth` | Date | |
| `national_id` | String (encrypted) | Authority-locked field — only KYC source can set |
| `passport_number` | String (encrypted) | Optional |
| `tax_id` | String | Optional |
| `originating_subsidiary_id` | ID | Which subsidiary first created this record |

### Product Summary Linkage Rules

DaVinci stores **summaries**, not full records:
- Loan summary: account number, product type, current balance, status (active/closed/delinquent), originating subsidiary
- Insurance summary: policy number, product type, premium status, status (active/lapsed/cancelled), originating subsidiary

Source of record for full loan/policy details remains in Core Banking / Policy Admin respectively. DaVinci summaries are kept current via event consumption (Event-Driven Synchronization capability).

### Deduplication Rule

On new customer creation or data change, DaVinci runs matching rules (name + DOB + National ID fuzzy match) to flag potential duplicates before creating a new record. Duplicates escalate to Customer Data Change Management (MERGE_DUPLICATES change type).

---

## NFRs

| NFR | Requirement |
|-----|-------------|
| `davinci_customer_id` immutability | Once assigned, the canonical ID never changes — even on merge operations |
| National ID encryption | National ID stored encrypted at rest |
| Authority-locked fields | National ID and DOB can only be set by their Rank 1 authority source (via Consolidation Engine) |

---

## Open Questions

- Is `davinci_customer_id` or National ID the primary lookup key for external system integration (e.g., when Onigiri calls DaVinci to look up a new applicant)? This is flagged as an unresolved decision in the ARCHITECTURE.md.
- What is the data retention policy for historical customer records (e.g., deceased customers)?
