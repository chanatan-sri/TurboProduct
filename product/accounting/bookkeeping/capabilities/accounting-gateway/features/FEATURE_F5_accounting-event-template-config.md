# FEATURE F5: Accounting Gateway — Accounting Event Template Config

**Feature ID**: F5
**POB Ticket**: POB-2192
**Type**: New Feature
**Priority**: P1
**Status**: Spec
**Parent Capability**: [Accounting Gateway](../CAPABILITY.md)
**Engineering Owner**: [TBC]
**Last Updated**: 2026-03-04

---

## User Story

As an accounting system administrator,
I want to configure Accounting Event templates that define the validation rules and double-entry mapping for each business event,
So that all accounting transaction requests are automatically validated and routed to the correct GL accounts without manual intervention.

## Job-to-be-done

Upstream systems submit transactions using only an event code — they don't know the GL accounts. The event config is the bridge that translates business events into valid double-entry journal entries automatically.

---

## System Context

**As-Is:** Accounting entries are mapped manually. No configurable event-to-ledger mapping layer exists.

**To-Be:** Each business event type has a configured template specifying DR/CR GL accounts, cost/profit center rules, sub-ledger rules, and reference field requirements. The gateway validates all incoming requests against this template before creating a journal entry.

**Components Impacted:** `master_accounting_event`, `accounting_config_event`, `accounting_config_field_requirements`, `accounting_event_references`, `master_reference_field`

---

## Data Specification

### Event Code Format

Pattern: `[Company prefix 1][Event category 3][Running number 3]` = 7 characters

| Company Prefix | Entity |
|---|---|
| `B` | NTB |
| `I` | NTBI |

| Category Code | Description |
|---|---|
| `COH` | Cash on Hand |
| `CAJ` | Cash on Hand — Adjustment |
| `PTC` | Petty Cash |
| `PAJ` | Petty Cash — Adjustment |
| `PAY` | Payment |
| `OPE` | Loan Operation |
| `BTC` | Manual / Test |

**Examples:** `BCOH001`, `BPTC014`, `BPAY001`, `BCAJ005`

### GL Resolution Logic

| Mechanism | When Used | Source |
|-----------|-----------|--------|
| Branch-derived | Most transaction lines | Branch code from Reference1 → lookup center from org unit master |
| Fixed HQ center `1002000000` | HQ-side lines | Hardcoded in event config |
| Fixed AM center `10050123` | AM cash transfer events | Hardcoded in event config |

### Sub-Ledger in Event Config

Only `2131000000` (Other AP - Related Companies) requires a sub-ledger across all events:
- `BCOH018`, `BCAJ018` → fixed vendor code `1020`
- `BPAY156` → fixed vendor code `1010`

### Reference Fields

- `Reference1` (branch code) is **mandatory for every event without exception**
- All other reference fields are event-specific

---

## Acceptance Criteria

**Scenario 1: Create a valid accounting event config**

Given an administrator with API access,
When a POST request is submitted with a valid Event Code in the `CCEE NNN` format, Event Name, Company Entity, DR GL, CR GL, and reference field definitions,
Then the system creates the accounting event config and it becomes available for transaction recording.

**Scenario 2: Event code format is enforced**

Given an administrator submitting a new event with a code that does not match the pattern (e.g., 6-character code or unrecognised company prefix),
When the request is processed,
Then the system rejects it with a format validation error.

**Scenario 3: Validate reference field requirements**

Given an accounting event configured with Reference1 as mandatory,
When a transaction request for that event is submitted without Reference1,
Then the system rejects the request with a clear validation error.

**Scenario 4: Event with optional sub-ledger**

Given an accounting event where the DR sub-ledger is optional,
When a transaction is submitted without a DR sub-ledger,
Then the system accepts the transaction and records it without the sub-ledger attribute on the DR side.

---

## Edge Cases & Out-of-Scope

| Scenario | Behaviour |
|----------|-----------|
| Duplicate event code | Reject — event codes must be unique |
| Deactivate an event with in-flight transactions | In-flight transactions unaffected; new transactions cannot use the deactivated event |
| Events `BCOH017`, `BPAJ031` (no date mapping) | These are manual-only — dates entered manually, no LOS date mapping |
| UI for event config management | Out of scope — Release 1 API-only |

---

## Dependencies

| Dependency | Type |
|-----------|------|
| F2 (GL Create) — event config references GL accounts | Blocking |
| F4/F6 (Centers) — event config references centers | Blocking |
| F8 (Upstream File Upload) — resolves GL from event config | Downstream |
| F7 (Single API Transaction) — validates against event config | Downstream |
