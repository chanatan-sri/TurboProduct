# CHANGELOG_018 — Risk Routing Open Question Resolutions

**Date**: 2026-05-25
**Author**: chanatan.sri@turbo.co.th
**Layer Affected**: Capability (Risk Assessment Engine, Application Approval) + Feature (Approval Routing Assignment)
**Branch**: loan-origination-with-cash
**Resolves**: Open questions from CHANGELOG_017

---

## Resolutions

| # | Question | Answer |
|---|----------|--------|
| 1 | Are risk levels 81–98 also auto-rejected? | **Yes.** 81–98 are system auto-rejected. Added as explicit row in all routing tables under `CA+` group. |
| 2 | DAM = Deputy Area Manager? | **Yes.** Full title confirmed: Deputy Area Manager. Documented as informational label in position column. |
| 3 | SAM = Senior Area Manager? | **Yes.** Full title confirmed: Senior Area Manager. Documented as informational label in position column. |
| 4 | Is CA+ (71–80) auto-rejection phase-specific? | **Yes.** Current phase: system auto-rejects. Future intent: route to human CRO approver once the CRO approval workflow is built. Noted in all routing tables. |

---

## Routing Mechanism Clarification

The system routes using `group_role`, not individual position titles. Position names (CO, SCO, BM, SBM, DAM, AM, SAM, CA, CRO) are informational labels that describe which org positions belong to each group. The worklist filter and role-match check at decision time both operate on `group_role` only. This clarification has been applied to all routing tables.

Updated column header in all tables from `"Approval Positions"` to `"Positions in group (informational)"` to make this explicit.

---

## Complete Routing Table (authoritative after this changelog)

| From RL | To RL | group_role | Positions in group (informational) | Outcome |
|---------|-------|------------|------------------------------------|---------|
| 1 | 10 | `SALE_BRANCH` | CO, SCO, BM, SBM | Routes to approver worklist |
| 11 | 20 | `SALE_AREA` | DAM (Deputy Area Manager), AM, SAM (Senior Area Manager) | Routes to approver worklist |
| 21 | 70 | `CA` | CA | Routes to approver worklist |
| 71 | 80 | `CA+` | CRO | System auto-rejects (phase-specific — future: human CRO) |
| 81 | 98 | `CA+` | CRO | System auto-rejects |
| 99 | 99 | *(system)* | GOD | System auto-rejects — policy violation, unconditional |
| — | 0 | *(default)* | — | System auto-rejects — unclassified |

---

## Documents Modified

| File | Change |
|------|--------|
| `capabilities/risk-assessment-engine/CAPABILITY.md` | Added 81–98 row; updated column header; clarified group_role routing mechanism |
| `capabilities/application-approval/CAPABILITY.md` | Added 81–98 row; updated column header; clarified group_role routing mechanism; updated strict role matching section |
| `capabilities/application-approval/features/FEATURE_approval-routing-assignment.md` | Added 81–98 row; full DAM/SAM titles added; updated column header; updated edge case range to 71–98 |
