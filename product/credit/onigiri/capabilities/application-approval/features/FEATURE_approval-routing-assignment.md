# Feature: Approval Routing Assignment

**Capability**: Application Approval — [CAPABILITY](../CAPABILITY.md)
**Product**: Onigiri — [PRODUCT](../../../PRODUCT.md)
**Engineering Owner**: TBD
**Status**: Spec

---

## User Story

As the **underwriting workflow engine**, I want to automatically assign a `required_approver_role` to each application entering `pending_approval` based on its aggregated risk level, so that the application is immediately routable to the correct human approver without any manual queue assignment.

## Job-to-be-Done

Eliminate the operational gap between "risk level is known" and "someone is accountable for the decision." The moment risk assessment completes, accountability must be established automatically and verifiably.

---

## Acceptance Criteria

1. When the workflow state transitions to `pending_approval`, the system reads `risk_level` from the Risk Assessment output stored on the application record.
2. The system maps `risk_level` to `required_approver_role` using the canonical table (see Business Rules).
3. `required_approver_role` is written to the application record atomically with the state transition to `pending_approval`. No application may enter `pending_approval` with a null or missing `required_approver_role`.
4. If `risk_level > 70` (i.e., falls in the 71–80, 99, or default 0 tiers), the system does **not** route to an approver — it transitions the application directly to `rejected` with reason `"Auto-decline: risk level {value}"`. The `pending_approval` state is bypassed entirely.
5. The assigned `required_approver_role` is recorded in the workflow audit log with: `risk_level`, `assigned_role`, `assignment_timestamp`.
6. `required_approver_role` is immutable after assignment — no in-place update path exists. Any role change requires a supervisor override (separate feature, out of scope here).

---

## Mapping Table

Risk level is evaluated as a range (`From RL` to `To RL`, inclusive). The value written to `required_approver_role` is the `group_role` — the system routes by group, not by individual position title. Any user carrying a matching `group_role` on their account may action the application.

| From RL | To RL | `required_approver_role` (group_role) | Positions in group (informational) | Action |
|---------|-------|--------------------------------------|------------------------------------|--------|
| 1 | 10 | `SALE_BRANCH` | CO, SCO, BM, SBM | Write role → enter `pending_approval` |
| 11 | 20 | `SALE_AREA` | DAM (Deputy Area Manager), AM, SAM (Senior Area Manager) | Write role → enter `pending_approval` |
| 21 | 70 | `CA` | CA | Write role → enter `pending_approval` |
| 71 | 80 | `CA+` | CRO | **Auto-reject** — bypass `pending_approval`; phase-specific (future: route to human CRO) |
| 81 | 98 | `CA+` | CRO | **Auto-reject** — bypass `pending_approval` |
| 99 | 99 | *(system)* | GOD | **Auto-reject** — policy violation, bypass `pending_approval` |
| — | 0 | *(default)* | — | **Auto-reject** — unclassified risk level, bypass `pending_approval` |

**Auto-rejection threshold**: Any application with `risk_level > 70` is rejected by the system without human review. This covers 71–80 (CA+, phase-specific), 81–98 (CA+), 99 (GOD/policy violation), and the unclassified default (0).

---

## Edge Cases and Error States

| Condition | Expected Behavior |
|-----------|-------------------|
| `risk_level` is absent from RAE output | Block state transition; raise alert; keep application in `risk_assessment` state |
| `risk_level` value is not in the mapping table (e.g., unexpected value) | Block state transition; raise alert; require engineering triage |
| Risk level > 70 (71–98, 99, or default 0) | Transition directly to `rejected`; do not enter `pending_approval` |
| State transition to `pending_approval` succeeds but role write fails | Roll back state transition (atomic guarantee); application remains in `risk_assessment` |

---

## Out of Scope

- Supervisor override / role reassignment
- Escalation when SLA expires
- Notification to the assigned approver (see Open Question #2 in CAPABILITY.md)

---

## Dependencies

- Risk Assessment Engine output (`risk_level`) must be written to the application record before this feature executes — guaranteed by RAE execution step in `risk_assessment` state.
- Underwriting Workflow state machine atomicity guarantee (see [Workflow State Machine Engine](../../underwriting-workflow/features/FEATURE_workflow-state-machine-engine.md)).
