# CHANGELOG_020 — Restructure Pre-Approval Open Question Resolutions

**Date**: 2026-05-25
**Author**: chanatan.sri@turbo.co.th
**Layer Affected**: Capability + Features (Restructure Pre-Approval)
**Branch**: loan-origination-with-cash
**Resolves**: Open questions 1, 2, 3, 5 from CHANGELOG_019

---

## Resolutions

| # | Question | Answer |
|---|----------|--------|
| 1 | Default expiry window? | **CA sets an explicit expiry date per plan at time of approval.** No system-default window — CA's judgment determines validity period per case. |
| 2 | Can sales reject CA's revision? | **Yes.** Sales can reject CA's revised terms → status `REVISION_REJECTED`. Contract is freed; a new plan may be submitted. |
| 3 | Multiple pre-approvals per contract? | **One active pre-approval per contract.** New submission allowed only after existing pre-approval reaches a closed status (`REJECTED`, `REVISION_REJECTED`, or `EXPIRED`). |
| 5 | Risk level fixed or RAE-dependent? | **RAE-dependent.** The pre-approval lowers the risk contribution of the restructuring-specific policy. The final risk level is the RAE aggregate (max across all policies) — not a guaranteed fixed value. |

*Question 4 (Core Banking contract lookup API/mechanism) remains open.*

---

## Lifecycle Update

Added `REVISION_REJECTED` as a new terminal status:

```
DRAFT → PENDING_CA_REVIEW → APPROVED
                          → REJECTED
                          → PENDING_REVISION → REVISION_CONFIRMED
                                             → REVISION_REJECTED  ← new
```

---

## Documents Modified

| File | Change |
|------|--------|
| `capabilities/restructure-pre-approval/CAPABILITY.md` | Resolved OQs 1–3, 5; added `REVISION_REJECTED` to lifecycle; added 1-per-contract rule; updated NFR for expiry; updated RAE integration section to remove fixed risk level claim |
| `features/FEATURE_pre-approval-decision.md` | Approve action now requires CA to enter expiry date; added `REVISION_REJECTED` to status diagram; updated downstream RAE impact wording |
| `features/FEATURE_pre-approval-request-tracker.md` | Added Reject Revision action; updated status badge list; added 1-per-contract edge case |
| `features/FEATURE_restructure-plan-builder.md` | Added AC #4 (1-per-contract block at contract selection); removed multi-plan reference; renumbered criteria |
