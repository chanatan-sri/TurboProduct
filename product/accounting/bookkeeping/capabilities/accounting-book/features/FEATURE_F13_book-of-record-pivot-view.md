# FEATURE F13: Accounting Book — Book of Record (Summary / Pivot View)

**Feature ID**: F13
**POB Ticket**: POB-2203
**Type**: New Feature
**Priority**: P2
**Status**: Spec
**Parent Capability**: [Accounting Book](../CAPABILITY.md)
**Engineering Owner**: [TBC]
**Last Updated**: 2026-03-04

---

## User Story

As an accounting team member,
I want to query a structured summary view of all accounting transactions and their SAP posting statuses,
So that I can monitor the state of the general ledger and produce the data needed for accounting work without needing direct access to raw journal tables.

## Job-to-be-done

The accounting team needs a queryable, formatted view of all transactions without touching raw database tables. The pivot view provides this as a pre-aggregated summary that reflects both transaction data and SAP posting outcomes.

---

## System Context

**As-Is:** No aggregated or formatted view of accounting data available for the accounting team.

**To-Be:** The system maintains a `pivot_view` table (RDS) providing a formatted, queryable summary including company code, event code, posting date, DR/CR account details, reference fields, SAP grouping document, and status. Updated asynchronously on two triggers.

**Components Impacted:** `pivot_view`, `pivot_view_refresh_log`, `journal_header`, `sap_transaction`

---

## Refresh Triggers

| Trigger | When |
|---------|------|
| New transaction saved | After any successful journal record creation (F7, F8, F10) |
| SAP result updated | After SAP posting result is processed (F16) |

Both refreshes are **asynchronous** — pivot view is eventually consistent, not real-time.

---

## Acceptance Criteria

**Scenario 1: New transaction appears in pivot view**

Given a new accounting transaction successfully created on the Accounting Book,
When the `pivot_view` is queried,
Then the transaction appears with correct company code, event code, account numbers, amount, and reference values.

**Scenario 2: JV transaction status reflected in pivot view**

Given an accounting transaction grouped into a JV batch and posted to SAP,
When the posting result is updated in the system,
Then the corresponding `pivot_view` record reflects the updated SAP status and group document reference.

**Scenario 3: Refresh log is maintained**

Given any refresh operation on the `pivot_view`,
When the refresh completes (successfully or with error),
Then a record is written to `pivot_view_refresh_log` with the refresh type, status, records processed, and timestamp.

---

## Edge Cases & Out-of-Scope

| Scenario | Behaviour |
|----------|-----------|
| Pivot view lag | Acceptable — async refresh means up to ~5 min lag is tolerated |
| Cancelled transactions (F19) | Reflected in pivot view with `Do Not Post` status after next refresh |
| UI display with manual refresh button | Out of scope — Release 1 is database table + query only |
| Real-time pivot view | Out of scope — async refresh is the design |

---

## Dependencies

| Dependency | Type |
|-----------|------|
| F12 (Journal Record) — source data | Blocking |
| F14 (SAP Outbound) — SAP status data | Related |
| F16 (Upload SAP Result) — triggers refresh on result update | Related |
| F19 (Cancel Transaction) — triggers refresh on cancellation | Related |

> **Note (Release 1):** Delivered as a database table for querying and export. UI display with manual refresh button planned for a future release.
