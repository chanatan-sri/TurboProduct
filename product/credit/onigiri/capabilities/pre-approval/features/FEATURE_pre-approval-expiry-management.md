# Feature: Pre-Approval Expiry Management

**Capability**: Pre-Approval — [CAPABILITY](../CAPABILITY.md)
**Product**: Onigiri
**Engineering Owner**: TBD
**Status**: Concept

---

## User Story

As a system, I want approved non-EasyPass pre-approvals to expire after a configurable period, so that outdated plans — based on campaign terms, eligibility conditions, or risk assessments that may have changed — cannot be converted into applications.

## Job-to-be-Done

An approved pre-approval is a snapshot in time. Campaign pricing, eligibility rules, and risk strategies can change. Expiry enforces a recheck cadence — ensuring the CO cannot convert a stale pre-approval into a Draft long after the conditions that justified it have changed.

---

## Acceptance Criteria

1. Every pre-approval that transitions to `approved` is assigned an expiry date.
2. The default expiry duration is system-configured (not hardcoded).
3. The approver may override the expiry date at the time of approving — setting a custom date shorter or longer than the system default.
4. On the expiry date, the system automatically transitions the pre-approval from `approved` to `expired` — no manual action required.
5. An `expired` pre-approval cannot be converted to a Draft application.
6. The expiry date and any override are recorded in the audit trail.
7. EasyPass pre-approvals do not carry an expiry date — this feature applies to non-EasyPass path only.

---

## Edge Cases and Error States

| Scenario | Expected Behaviour |
|---|---|
| Approver sets expiry date in the past | Validation rejects — expiry date must be in the future at time of approval. |
| CO attempts to convert an `expired` pre-approval | Convert action blocked. CO informed pre-approval has expired and must submit a new request. |
| Pre-approval transitions to `converted` before expiry date | Expiry no longer applies — pre-approval is in terminal `converted` state. |
| System fails to run expiry job on expiry date | Pre-approval remains `approved` until job runs. Must not allow conversion after expiry date even if state transition is delayed — expiry date check is enforced at conversion time as a secondary guard. |

---

## Dependencies

- Underwriting Workflow engine (Topology D) must support `expired` state and automatic system-triggered transitions.
- System configuration must expose a default expiry duration parameter.
