# FEATURE F6: COA Management — Real-Time Cost & Profit Center Sync from Upstream System

**Feature ID**: F6
**POB Ticket**: POB-2193
**Type**: New Feature
**Priority**: P1
**Status**: Spec
**Parent Capability**: [COA Management](../CAPABILITY.md)
**Engineering Owner**: [TBC]
**Last Updated**: 2026-03-04

---

## User Story

As the Bookkeeping system,
I want to create and update cost and profit center records in real time when the upstream system creates or modifies a branch,
So that branch-level accounting attribution is always current without manual data entry or batch delays.

## Job-to-be-done

Branches are created and updated frequently in the upstream LOS. Without real-time sync, accounting transactions for new branches would fail validation because their centers don't yet exist in Bookkeeping.

---

## System Context

**As-Is:** Cost and profit center records for branches must be created manually after a branch is opened or updated upstream.

**To-Be:** LOS calls the Bookkeeping organization unit API in real time whenever a branch is created or updated. Bookkeeping creates or updates the corresponding record immediately.

**Components Impacted:** `master_organizational_unit`, `master_unit_type`, `master_business_units`, LOS integration layer

---

## API Specification

**Endpoint:** `POST /api/bookkeeping/organization-units`
**Base URL:** `https://bks-api.uat01.ntbx.tech`
**Authentication:** `userId` header

### Request Headers

| Header | Required | Description |
|--------|----------|-------------|
| `userId` | Required | ID of the user or system triggering the request |
| `Content-Type` | Required | Must be `application/json` |

### Request Body

| Field | Required | Type | Description |
|-------|----------|------|-------------|
| `unitCode` | Required | String | Unique branch code |
| `unitName` | Required | String | Display name of the branch |
| `unitTypeId` | Required | String | Type identifier (e.g., `"1"` = NTB Branch) |
| `unitCenterDetail` | Required | Array | List of cost/profit center assignments |
| `unitCenterDetail[].centerTypeId` | Required | Integer | `1` = Cost Center, `2` = Profit Center |
| `unitCenterDetail[].centerCode` | Required | String | The center code to assign |

### Example Request

```bash
curl --location 'https://bks-api.uat01.ntbx.tech/api/bookkeeping/organization-units' \
--header 'userId: 704' \
--header 'Content-Type: application/json' \
--data '{
    "unitCode": "UATTEST01",
    "unitName": "Branch Test UAT01-1",
    "unitTypeId": "1",
    "unitCenterDetail": [
        { "centerTypeId": 1, "centerCode": "10051110" },
        { "centerTypeId": 2, "centerCode": "1005111000" }
    ]
}'
```

---

## Acceptance Criteria

**Scenario 1: New branch is created via API**

Given a valid API request with a `unitCode` that does not yet exist in Bookkeeping,
When the upstream system calls `POST /api/bookkeeping/organization-units`,
Then the system creates a new organizational unit record with the provided unit code, name, type, and center codes, and returns a success response immediately.

**Scenario 2: Existing branch details are updated via API**

Given a valid API request with a `unitCode` that already exists,
When the upstream system calls `POST /api/bookkeeping/organization-units`,
Then the system updates the existing record with the new values and returns a success response.

**Scenario 3: Request with missing required field is rejected**

Given an API request where `unitCode` or `unitName` is absent,
When the request is received,
Then the system returns a validation error without creating or modifying any record.

**Scenario 4: Request is processed in real time**

Given a branch creation event in the upstream system,
When the upstream system dispatches the API call,
Then the organizational unit record is available in Bookkeeping immediately upon successful response — no batch delay.

---

## Edge Cases & Out-of-Scope

| Scenario | Behaviour |
|----------|-----------|
| Upstream sends same `unitCode` twice (duplicate call) | Upsert — update the existing record |
| `unitCenterDetail` is empty array | Reject — at least one center must be provided |
| Center sync for non-branch entities | Out of scope for Release 1 |

---

## Dependencies

| Dependency | Type |
|-----------|------|
| F4 (Cost/Profit Center Create) — this feature automates what F4 does manually | Related |
| LOS system must call this API on branch create/update | External dependency |
| F7, F8, F10 (Gateway) validate center codes against this master | Downstream |
