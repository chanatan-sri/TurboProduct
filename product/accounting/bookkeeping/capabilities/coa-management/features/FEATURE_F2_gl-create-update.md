# FEATURE F2: COA Management — General Ledger Create & Update

**Feature ID**: F2
**POB Ticket**: POB-2199
**Type**: New Feature
**Priority**: P1
**Status**: Spec
**Parent Capability**: [COA Management](../CAPABILITY.md)
**Engineering Owner**: [TBC]
**Last Updated**: 2026-03-04

---

## User Story

As an accounting system administrator,
I want to create, update, and retire General Ledger (GL) accounts via API,
So that the chart of accounts remains accurate, non-duplicated, and traceable for all transaction recording.

## Job-to-be-done

GL accounts are the atomic unit of all journal entries. Without a validated, deduplicated GL master, transactions can be posted to wrong or non-existent accounts, creating financial reporting errors and audit exposure.

---

## System Context

**As-Is:** GL accounts are managed externally. No system-level validation prevents duplicate GL numbers or references to retired FS Groups.

**To-Be:** API enforces GL creation rules. GL numbers are unique, FS Group references are validated, and configuration history between GL and FS Group is recorded.

**Components Impacted:** `master_general_ledger`, `master_fs_group`, `master_sub_ledger_type`, `master_tracking_dimension`

---

## Data Specification

### GL Account Number Format

10-digit numeric: `[type 1][major group 3][sub-group 2][item 4]`

| Leading Digit | Account Type |
|---|---|
| 1 | Asset |
| 2 | Liability |
| 4 | Revenue / Income |
| 5 | Expense |
| 9 | Suspense / Off-balance |

### Examples from Live GL Master

| Account Number | Name | FS Group | Center Type |
|---|---|---|---|
| 1110000000 | Cash on Hand | A101 | Profit Center |
| 1110100000 | Petty Cash | A101 | Profit Center |
| 2131000000 | Other AP - Related Companies | L104 | Profit Center |
| 5120000000 | Seizing Expense | PL218 | Cost Center |
| 5211217000 | Bank Fee | PL219 | Cost Center |

---

## Acceptance Criteria

**Scenario 1: Create a valid GL account**

Given a user with API access and a valid active FS Group code,
When a POST request is submitted with all required fields (FS Group Code, Account Number, Account Name TH, Account Name EN, Sub-ledger Type),
Then the system creates the GL account and returns a success response.

**Scenario 2: Prevent duplicate GL number**

Given an existing GL account with number "1110100000",
When a new request attempts to create a GL account with the same number,
Then the system rejects the request with an appropriate validation error.

**Scenario 3: Prevent reference to inactive FS Group**

Given an FS Group that has been retired,
When a request attempts to create a GL account under that group,
Then the system rejects the request.

**Scenario 4: Enforce 10-digit account number format**

Given a user with API access,
When a POST request is submitted with an account number that is not exactly 10 digits (e.g., 9 digits or alphanumeric),
Then the system rejects the request with a format validation error.

---

## Edge Cases & Out-of-Scope

| Scenario | Behaviour |
|----------|-----------|
| Retire a GL account with existing transactions | Allowed — existing transactions unaffected; new transactions cannot reference the retired GL |
| Change FS Group of an existing GL | Configuration history must be recorded |
| Duplicate GL number across entities | Not allowed — GL numbers are unique system-wide |
| Self-service UI | Out of scope — Release 1 API-only |

---

## Dependencies

| Dependency | Type |
|-----------|------|
| F1 (FS Group) must exist and be active | Blocking |
| F3 (Sub-Ledger Type Assignment) extends this feature | Downstream |
| F5 (Event Config) references GL accounts | Downstream |
| F7, F8, F10 (Gateway) validate GL against this master | Downstream |

> **Note (Release 1):** API tool only. Must record configuration history between GL and FS Group.
