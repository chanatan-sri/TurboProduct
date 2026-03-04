# FEATURE F12: Accounting Book — Transaction Journal Record (Raw Data)

**Feature ID**: F12
**POB Ticket**: POB-2201
**Type**: New Feature
**Priority**: P1
**Status**: Spec
**Parent Capability**: [Accounting Book](../CAPABILITY.md)
**Engineering Owner**: [TBC]
**Last Updated**: 2026-03-04

---

## User Story

As the Bookkeeping system,
I want a high-integrity raw data store for every accounting transaction,
So that there is a complete, immutable audit trail of all double-entry records including sub-ledger and cost/profit center attributes.

## Job-to-be-done

Every validated transaction must be permanently recorded with full detail. This is the system of record — all downstream processes (pivot view, SAP connector) read from here.

---

## System Context

**As-Is:** No centralized, validated raw journal store exists.

**To-Be:** Every successfully validated accounting transaction is persisted with full DR/CR GL detail, optional sub-ledger and cost/profit center attributes, reference data, posting date, doc date, and active status flag.

**Components Impacted:** `journal_header`, `journal_line`, `journal_line_attributes`, `journal_header_references`

---

## Record Structure

| Table | Role |
|-------|------|
| `journal_header` | 1 per transaction — event code, posting date, doc date, entity, active status (`is_active`) |
| `journal_line` | 2 per transaction (DR + CR) — GL account, accounting type, amount |
| `journal_line_attributes` | Optional — cost center, profit center, sub-ledger (vendor/customer code) per line |
| `journal_header_references` | Reference1–Reference6 values per transaction |

---

## Acceptance Criteria

**Scenario 1: Transaction stored with full double-entry detail**

Given a successfully validated accounting transaction,
When the transaction is created,
Then the system persists a `journal_header` linked to the accounting event, and two `journal_line` records (one DR, one CR) each with the correct GL account, accounting type, and amount.

**Scenario 2: Reference data is stored**

Given a transaction containing values for Reference1 and Reference2,
When the transaction is created,
Then `journal_header_references` records are created for each reference field with the corresponding values.

**Scenario 3: Active status defaults to true**

Given a newly created accounting transaction,
When the record is stored,
Then `is_active` defaults to `true` and can be updated to `false` by an authorized user (via F19 Cancel).

---

## Edge Cases & Out-of-Scope

| Scenario | Behaviour |
|----------|-----------|
| DR and CR amounts must balance | Enforced — transaction rejected if DR ≠ CR |
| Deleting a journal record | Not supported — records are immutable; cancellation sets `is_active = false` |
| Retroactive modification of a record | Not supported — create a correcting entry instead |

---

## Dependencies

| Dependency | Type |
|-----------|------|
| F7, F8, F10 (Gateway) — create journal records after validation | Upstream |
| F13 (Pivot View) — reads from journal records | Downstream |
| F14 (SAP Outbound) — creates `sap_transaction` for each journal record | Downstream |
| F19 (Cancel) — sets `is_active = false` on this record | Downstream |
