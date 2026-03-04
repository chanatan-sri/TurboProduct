# FEATURE F3: COA Management — GL Sub-Ledger Type Assignment

**Feature ID**: F3
**POB Ticket**: POB-2194
**Type**: New Feature
**Priority**: P2
**Status**: Spec
**Parent Capability**: [COA Management](../CAPABILITY.md)
**Engineering Owner**: [TBC]
**Last Updated**: 2026-03-04

---

## User Story

As an accounting system administrator,
I want to assign a sub-ledger type to each GL account that requires counterparty tracking,
So that the system can enforce at transaction entry time whether a Customer or Vendor code must accompany postings to that account.

## Job-to-be-done

Reconciliation accounts (AP/AR) require a counterparty code on every posting. Without a system-level flag, transactions can be posted to reconciliation accounts without a sub-ledger code, making reconciliation impossible.

---

## System Context

**As-Is:** No system-level flag on GL accounts indicates whether a counterparty code is required.

**To-Be:** Each GL account carries a Sub-ledger Type (`Customer`, `Vendor`, or `None`). The counterparty code value itself is configured at the Accounting Event level (F5) — not here.

**Components Impacted:** `master_sub_ledger_type`, `master_general_ledger`, `journal_line`, `journal_line_attributes`

---

## Data Specification

### Sub-Ledger Type Values

| Sub-Ledger Type | Meaning | Example GL |
|---|---|---|
| `Vendor` | Posting requires a vendor counterparty code | `2131000000` — Other AP - Related Companies |
| `Customer` | Posting requires a customer counterparty code | `1153100000` — Trade AR - Related Companies |
| `None` | No counterparty required | `1110000000` — Cash on Hand |

> The counterparty code value (e.g., `1020`, `1010`) is NOT stored in the GL master. It is defined as a fixed value within each Accounting Event config (F5) and written to `journal_line_attributes` at transaction time.

---

## Acceptance Criteria

**Scenario 1: Assign Sub-Ledger Type during GL creation**

Given a user with API access,
When a GL account is created with Sub-Ledger Type set to `Vendor`,
Then the system records the type and the account is flagged as requiring a vendor code on every posting.

**Scenario 2: Transaction posting to a Vendor GL without a counterparty code**

Given a GL account with Sub-Ledger Type = `Vendor`,
When an accounting transaction is recorded against that account without a vendor counterparty code,
Then the system rejects the journal line with a validation error indicating the counterparty code is required.

**Scenario 3: Transaction posting to a None GL without a counterparty code**

Given a GL account with Sub-Ledger Type = `None`,
When an accounting transaction is recorded against that account without a counterparty code,
Then the system accepts the journal line normally.

---

## Edge Cases & Out-of-Scope

| Scenario | Behaviour |
|----------|-----------|
| Sub-ledger type change on existing GL | Allowed — but existing transactions are not retroactively affected |
| Counterparty code value management | Out of scope — managed in F5 (Event Config) |
| F10 manual path: user provides sub-ledger value for a `None` GL | Silently ignored — no error |

---

## Dependencies

| Dependency | Type |
|-----------|------|
| F2 (GL Create) — sub-ledger type is set on the GL account | Blocking |
| F5 (Event Config) — counterparty code value is set here | Related |
| F7, F8, F10 (Gateway) — enforce sub-ledger requirement at posting time | Downstream |
