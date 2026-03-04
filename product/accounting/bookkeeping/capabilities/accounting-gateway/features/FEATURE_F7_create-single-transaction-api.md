# FEATURE F7: Accounting Gateway — Create Single Accounting Transaction (API)

**Feature ID**: F7
**POB Ticket**: POB-2205
**Type**: New Feature
**Priority**: P1
**Status**: Spec
**Parent Capability**: [Accounting Gateway](../CAPABILITY.md)
**Engineering Owner**: [TBC]
**Last Updated**: 2026-03-04

---

## User Story

As an upstream business system (e.g., AMS, Cash Reconcile),
I want to submit a single accounting transaction request via API,
So that the Bookkeeping system records the validated journal entry in real time.

## Job-to-be-done

Real-time business events (e.g., a single cash disbursement) need to be recorded immediately as journal entries. Batch file upload is not suitable for time-sensitive single transactions.

---

## System Context

**As-Is:** Business systems submit accounting data via file-based or manual processes with no real-time validation.

**To-Be:** A dedicated API endpoint accepts single transaction requests, validates against the Accounting Event config, creates the double-entry journal record, and returns the result immediately.

**Components Impacted:** `journal_header`, `journal_line`, `journal_line_attributes`, `journal_header_references`, `master_accounting_event`, `log_request`, `log_response`

---

## Acceptance Criteria

**Scenario 1: Valid transaction request is processed**

Given an active accounting event config for the submitted event code,
When an API request is submitted with a valid event code, amount, posting date, doc date, and all required reference fields,
Then the system creates a journal header and journal line records (DR and CR), returns a success response, and logs the request.

**Scenario 2: Transaction fails event validation**

Given an event config requiring Reference1 as mandatory,
When a request is submitted for that event without Reference1,
Then the system rejects the request, returns a structured error response with an error code, and does not create any journal records.

**Scenario 3: Request log is created regardless of outcome**

Given any inbound API request to the accounting gateway,
When the request is received,
Then a log entry is created in `log_request` before validation begins, regardless of the validation outcome.

---

## Edge Cases & Out-of-Scope

| Scenario | Behaviour |
|----------|-----------|
| Event code does not exist | Reject with error — event not found |
| Amount = 0 or negative | Reject — amount must be > 0 |
| Duplicate submission (same payload twice) | Idempotency behaviour TBC — see open items |
| SAP Record flag handling | GL resolution from event config determines the SAP Record flag |

---

## Dependencies

| Dependency | Type |
|-----------|------|
| F5 (Event Config) — validates event code and resolves GL | Blocking |
| F12 (Journal Record) — stores the created transaction | Downstream |
| F14 (SAP Outbound) — creates `sap_transaction` record after save | Downstream |

---

## Open Items

| # | Question | Status |
|---|----------|--------|
| 1 | Full API request/response schema not yet provided — acceptance criteria for API contracts are incomplete | Open — pending API spec doc |
| 2 | Idempotency key — is duplicate submission detection enforced? | Open |
