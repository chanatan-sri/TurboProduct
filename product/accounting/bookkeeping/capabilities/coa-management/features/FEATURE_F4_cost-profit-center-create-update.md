# FEATURE F4: COA Management — Cost & Profit Center Create & Update

**Feature ID**: F4
**POB Ticket**: POB-2195
**Type**: New Feature
**Priority**: P1
**Status**: Spec
**Parent Capability**: [COA Management](../CAPABILITY.md)
**Engineering Owner**: [TBC]
**Last Updated**: 2026-03-04

---

## User Story

As an accounting system administrator,
I want to create and maintain cost center and profit center records,
So that accounting transactions can be attributed to the correct organizational units for management reporting.

## Job-to-be-done

Every transaction must be attributed to a branch or cost center. Without a validated center master, transactions are posted without proper organizational attribution, breaking management reporting and cost allocation.

---

## System Context

**As-Is:** Cost and profit center data is not managed within Bookkeeping. No validation when transactions reference cost/profit centers.

**To-Be:** The system maintains a structured repository of cost/profit centers. Each GL account's master config determines whether it maps to a Profit Center or Cost Center.

**Components Impacted:** `master_organizational_unit`, `master_unit_type`, `master_business_units`

---

## Acceptance Criteria

**Scenario 1: Create a cost/profit center unit**

Given a user with API access,
When a request is submitted with Unit Code, Unit Name, Unit Type, and at least one of Cost Center Number or Profit Center Number,
Then the system creates the organizational unit record.

**Scenario 2: Reject duplicate unit code**

Given an existing unit with code "10050121",
When a request submits a new unit with the same code,
Then the system returns an error: "Cost Center Number is duplicated".

**Scenario 3: Disable a unit**

Given an active cost center unit,
When a disable request is submitted via API,
Then the unit is marked inactive and cannot be referenced in new accounting transactions.

---

## Edge Cases & Out-of-Scope

| Scenario | Behaviour |
|----------|-----------|
| Unit code must be unique | Enforced — duplicates rejected |
| Disable a unit with in-flight transactions | Allowed — existing transactions unaffected; new transactions cannot reference the disabled unit |
| Center sync from upstream (LOS) | Covered by F6 — this feature is for manual creation only |
| Center type determination (profit vs cost) | Determined by the GL account's master config at transaction time, not stored on the center record |

---

## Dependencies

| Dependency | Type |
|-----------|------|
| F6 (Real-Time Sync) extends this feature for automated creation from upstream | Related |
| F7, F8, F10 (Gateway) validate center codes against this master | Downstream |
