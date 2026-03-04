# Capability: Consent-Based Data Visibility

**Product**: DaVinci — [PRODUCT](../../PRODUCT.md)
**Portfolio**: Platform
**Product Owner**: TBD (Platform PO)
**Status**: 📝 Draft — @FEATURE decomposition pending
**Last Updated**: 2026-03-04

---

## Business Function

Enforce Thai PDPA consent rules at the data access layer, ensuring that customer information is only visible to users whose subsidiary has obtained the customer's explicit consent. This is entity-level consent gating — not role-based security.

## Why It Exists (First Principles)

The company has multiple legal subsidiaries. A customer who purchased insurance from Subsidiary A has **not** necessarily consented to share their data with the parent lending entity (Subsidiary H). Thai PDPA mandates that data sharing across legal entities requires explicit customer consent. This cannot be enforced by role-based access control (RBAC) — it requires a consent registry as a first-class entity.

---

## Feature Inventory

| Feature | Status | Description |
|---------|--------|-------------|
| Consent Registry | Draft | Per customer-subsidiary consent records: type, granted_at, revoked_at, channel, document_ref |
| API-Level Visibility Filter | Draft | All DaVinci APIs filter results based on requesting user's subsidiary and applicable consent records |
| Directed Consent Model | Draft | A→B consent does not imply B→A; each direction requires separate explicit consent |
| Consent Revocation | Draft | Upon revocation, visibility immediately restricted; data not deleted but inaccessible to non-consented entity |
| Consent Audit Trail | Draft | All consent grants, revocations, and access attempts logged immutably |

---

## Business Rules

### Visibility Rules

| Scenario | Access |
|----------|--------|
| User from Subsidiary H queries customer data | Sees only data from Subsidiary H by default |
| Customer has A→H consent granted | User from Subsidiary H can see Subsidiary A's product summaries for that customer |
| Consent is A→H only | User from Subsidiary A cannot see Subsidiary H's data (directed model) |
| Consent revoked | Immediately restricted; existing data inaccessible to non-consented entity |

### Consent Record Fields

| Field | Description |
|-------|-------------|
| `customer_id` | DaVinci customer ID |
| `granting_subsidiary_id` | Subsidiary whose data is being shared (A in A→H) |
| `receiving_subsidiary_id` | Subsidiary that gains access (H in A→H) |
| `consent_type` | e.g., `CROSS_ENTITY_DATA_SHARING` |
| `granted_at` | ISO timestamp of consent grant |
| `revoked_at` | ISO timestamp of revocation (null if active) |
| `consent_channel` | branch / app / web |
| `consent_document_ref` | Reference to signed consent document |

### API Enforcement Rule

All DaVinci APIs must filter results based on consent. There is no "see everything" bypass outside of a designated compliance/audit role. Every API call's context (requesting user's subsidiary) determines which product summaries are returned.

---

## Regulatory Context

| Regulation | Requirement | DaVinci Response |
|------------|-------------|------------------|
| PDPA (Thailand) | Cross-entity data sharing requires explicit consent | Consent Registry + API-level visibility filtering |
| Debt Collection Act | Cross-product contact coordination | Combined view enabled only with consent |

---

## NFRs

| NFR | Requirement |
|-----|-------------|
| Real-time revocation enforcement | Consent revocation must take effect within seconds — no cache lag |
| Audit completeness | All consent grants, revocations, and access attempts logged immutably |
| Default restricted | Without explicit consent, visibility is restricted (fail-safe default) |

---

## Open Questions

- Does the "compliance/audit role" bypass consent filtering for regulatory investigations? What is the governance process for invoking this bypass?
- How is initial consent collected? Is there a customer-facing consent portal, or is it always collected by branch staff?
