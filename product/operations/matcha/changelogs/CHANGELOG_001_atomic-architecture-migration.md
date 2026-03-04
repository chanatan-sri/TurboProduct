# Changelog: Atomic Architecture Migration

**Product**: Matcha (Document Verification Service)
**Portfolio**: Operations
**Changelog #**: 001
**Layer Affected**: Portfolio / Product / Capability
**Date**: 2026-03-04
**Author**: Claude (AI-assisted migration)

---

## What Changed

### Directory Restructure
- **Moved**: `product/matcha/` → `product/operations/matcha/`
- Git history fully preserved via `git mv` (all ATLAS.md, ARCHITECTURE.md, and artifacts tracked as renames)
- Working directory now lives under the `operations/` portfolio subdirectory in alignment with CLAUDE.md file system convention

### Files Created

| File | Layer | Description |
|------|-------|-------------|
| `product/operations/PORTFOLIO.md` | Portfolio | Operations portfolio definition: investment thesis (Matcha + Wasabi + Sensei), shared dependency on DaVinci, cross-product OKRs, Matcha's role as shared service for all portfolios |
| `product/operations/matcha/PRODUCT.md` | Product | Full product spec: problem statement, value proposition, IS/IS NOT boundary, 6-capability registry, task lifecycle state machine (Mermaid), integration map, technology stack, KPIs |
| `product/operations/matcha/capabilities/universal-document-verification-engine/CAPABILITY.md` | Capability | Domain-agnostic task lifecycle: PENDING → IN_PROGRESS → COMPLETED → PENDING_REVIEW; Solomon QA distribution |
| `product/operations/matcha/capabilities/flexible-logic-configuration/CAPABILITY.md` | Capability | DB-driven data items + policy items; per-document correct/incorrect decisions; task outcomes: APPROVED / RETURNED / REFERRED |
| `product/operations/matcha/capabilities/async-car-check-integration/CAPABILITY.md` | Capability | Hybrid sync/async car check via Haibara; bypass with cut-off; PENDING_REVIEW re-entry on late data |
| `product/operations/matcha/capabilities/safety-integrity-guardrails/CAPABILITY.md` | Capability | Dual-hash change detection (context_hash / decision_hash); visual Revised alert; pre-filled decision clearing; immutable TaskCompletionEvent audit trail |
| `product/operations/matcha/capabilities/re-flow/CAPABILITY.md` | Capability | Same surrogate key versioning; smart result copy for unchanged documents (matching context_hash); is_changed flag; previous_document_id linking |
| `product/operations/matcha/capabilities/ai-first-verification/CAPABILITY.md` | Capability | Verification Router: routes based on Wasabi AI results; Uncertain → Human QA; Confident (≥99.99%) → Auto-Complete; 0.1% spot-check sampling |
| `product/operations/matcha/artifacts/research/.gitkeep` | Infrastructure | Scaffold: artifacts/research/ directory (was missing; diagrams/ already existed) |

### Files Kept (Unchanged)
- `product/operations/matcha/ATLAS.md` — primary source of truth
- `product/operations/matcha/ARCHITECTURE.md` — technical specifications (note: relative cross-reference paths may need correction post-mv)
- `product/operations/matcha/artifacts/diagrams/task_state_machine.mermaid` — preserved
- `product/operations/matcha/artifacts/.gitkeep` — preserved

---

## Rationale

Matcha is a shared service used across multiple portfolios (Credit via Onigiri, future Insurance, future KYC). Housing it under the `operations/` portfolio correctly reflects its cross-portfolio role as a universal document verification engine. The Operations portfolio groups Matcha (verification engine), Wasabi (AI routing layer), and Sensei (branch orchestration) — all three are operational infrastructure consumed by business-facing portfolios.

---

## Decision Log

| Decision | Rationale |
|----------|-----------|
| Portfolio: Operations | Matcha provides a shared operational service; it is not specific to Credit or Insurance. Operations portfolio is the correct home for cross-portfolio operational infrastructure |
| ATLAS.md and ARCHITECTURE.md kept in-place | Source of truth during transition; CAPABILITY.md files are extractions |
| FEATURE_*.md deferred | Requires engineering owner assignment and testable acceptance criteria |
| `git mv` used, not OS `mv` | Preserves full git history for all files |
| Broken path note | `ARCHITECTURE.md` contains relative cross-reference `../02-Credit_Approval/Architecture-02-Credit-Approval.md` — this path is now one level deeper after the `git mv`. Flagged as a known issue; path correction is a separate cleanup commit. |

---

## Known Issues

- `ARCHITECTURE.md` contains a relative path `../02-Credit_Approval/Architecture-02-Credit-Approval.md` that is now invalid after the `git mv`. This requires a path correction in a follow-up commit.

---

## Links

- [PRODUCT.md](../PRODUCT.md)
- [PORTFOLIO.md](../../PORTFOLIO.md)
- [PORTFOLIO_CATALOG.md](../../../PORTFOLIO_CATALOG.md)
- [ATLAS.md](../ATLAS.md) (source)
- [ARCHITECTURE.md](../ARCHITECTURE.md) (source)
