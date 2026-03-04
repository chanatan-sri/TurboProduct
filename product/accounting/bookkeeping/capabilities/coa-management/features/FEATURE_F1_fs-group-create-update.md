# FEATURE F1: COA Management — FS Group Create & Update

**Feature ID**: F1
**POB Ticket**: POB-2197
**Type**: New Feature
**Priority**: P1
**Status**: Spec
**Parent Capability**: [COA Management](../CAPABILITY.md)
**Engineering Owner**: [TBC]
**Last Updated**: 2026-03-04

---

## User Story

As an accounting system administrator,
I want to create, update, and retire FS Groups via API,
So that General Ledger accounts can be correctly classified under the right financial statement groupings for reporting and compliance.

## Job-to-be-done

The accounting team needs a governed, validated Chart of Accounts hierarchy. FS Groups are the top-level classification layer. Without them, GL accounts cannot be assigned to the correct financial statement categories (Assets, Liabilities, etc.), making financial reporting impossible.

---

## System Context

**As-Is:** FS group configuration is maintained manually outside the system. No API or validation layer enforces FS group integrity.

**To-Be:** The system exposes an API for creating and managing FS Groups. Each group is validated at creation time. Inactive groups cannot be referenced by new GL accounts.

**Components Impacted:** `master_fs_group`, `master_fs_group_type`, `master_general_ledger`

---

## Data Specification

### FS Group Type Catalogue

| ID | Type Code | Name (EN) | Release 1 Status |
|----|-----------|-----------|-----------------|
| 1 | `A` | Assets | Active — 36 groups |
| 2 | `L` | Liabilities | Active — 17 groups |
| 3 | `SE` | Shareholders' Equity | Active — 7 groups |
| 4 | `R` | Revenue | Defined — no groups assigned |
| 5 | `E` | Expense | Defined — no groups assigned |
| 6 | `PL` | Profit & Loss (incl. OCI) | Active — 54 groups |

### FS Group Code Format

`[Type prefix][Numeric suffix]` — e.g., `A101`, `L104`, `SE105`, `PL219B`, `OCI101`
- Prefix must align with the FS Group Type code
- Numeric suffix has no fixed length; may include trailing letter (e.g., `A104A`)
- OCI groups use `OCI` prefix but map to `fs_group_type_id = 6` (PL)

---

## Acceptance Criteria

**Scenario 1: Create a valid FS Group**

Given a user with appropriate API access,
When a POST request is submitted with all required fields (FS Group Type, Group Code, Group Name TH, Group Name EN),
Then the system creates the FS Group record and returns a success response with the new group ID.

**Scenario 2: Create FS Group with missing required field**

Given a user with API access,
When a POST request is submitted with a missing required field (e.g., Group Name EN is absent),
Then the system rejects the request and returns a descriptive error message (e.g., "Group Name - EN is required").

**Scenario 3: Retire an FS Group**

Given an existing active FS Group,
When a deactivation request is submitted via API,
Then the FS Group is marked inactive and can no longer be referenced when creating new GL accounts.

---

## Edge Cases & Out-of-Scope

| Scenario | Behaviour |
|----------|-----------|
| Duplicate FS Group code | Reject — codes must be unique |
| Retire a group with active GL accounts referencing it | Allowed — existing GLs are unaffected; only new GL creation is blocked |
| FS Group Type R and E (Revenue/Expense) | Defined in schema but no groups assigned in Release 1 — creation via API should still work |
| Self-service UI for accounting team | Out of scope — Release 1 is API-only (dev team only) |

---

## Dependencies

| Dependency | Type |
|-----------|------|
| `master_fs_group_type` must be seeded | Blocking — types must exist before groups can be created |
| F2 (GL Create) depends on this feature | Downstream — GL accounts reference FS Group codes |

---

## NFRs

- All create/update/retire operations must be logged with timestamp and actor
- Changes must be immediately visible to GL creation validation after save

> **Note (Release 1):** API tool only. Accounting team cannot self-manage FS Groups.
