# Changelog: Atomic Architecture Migration

**Product**: Wasabi (AI Document Verification Service)
**Portfolio**: Operations
**Changelog #**: 001
**Layer Affected**: Portfolio / Product / Capability
**Date**: 2026-03-04
**Author**: Claude (AI-assisted migration)

---

## What Changed

### Directory Restructure
- **Moved**: `product/wasabi/` → `product/operations/wasabi/`
- Git history fully preserved via `git mv`
- Working directory now lives under the `operations/` portfolio subdirectory in alignment with CLAUDE.md file system convention

### Files Created

| File | Layer | Description |
|------|-------|-------------|
| `product/operations/wasabi/PRODUCT.md` | Product | Full product spec: problem statement, value proposition, IS/IS NOT boundary (stateless; Onigiri stores results Phase 1; Matcha stores results Phase 2), 4-capability registry, two-phase value delivery sequence diagram (Mermaid), three core questions, architectural properties table, KPIs |
| `product/operations/wasabi/capabilities/document-quality-assessment/CAPABILITY.md` | Capability | Pre-extraction image quality gate: blur, lighting, truncation, obstruction, watermarks; quality score 0–100; unusable images skip remaining stages |
| `product/operations/wasabi/capabilities/document-type-classification/CAPABILITY.md` | Capability | LLM identifies document type from image; compares to expected documentTypeKey; mismatch or confidence < 99.99% → flag for human review |
| `product/operations/wasabi/capabilities/instruction-verification/CAPABILITY.md` | Capability | Data items: LLM extracts + compares against system data. Policy items: LLM evaluates check_description as natural-language instruction. Per-item results with confidence and reasoning |
| `product/operations/wasabi/capabilities/report-assembly/CAPABILITY.md` | Capability | Aggregates all stage results into a per-document verification report; overall human_review_required flag; idempotent by requestId |
| `product/operations/wasabi/artifacts/diagrams/.gitkeep` | Infrastructure | Scaffold: artifacts/diagrams/ directory (was missing) |
| `product/operations/wasabi/artifacts/research/.gitkeep` | Infrastructure | Scaffold: artifacts/research/ directory (was missing) |

### Files Kept (Unchanged)
- `product/operations/wasabi/ATLAS.md` — primary source of truth

---

## Rationale

Wasabi is Matcha's AI routing layer — it processes document images and returns structured verification reports that Matcha uses to route tasks to auto-complete or human QA. Grouping Wasabi under `operations/` alongside Matcha correctly reflects this tight coupling. Wasabi is stateless and model-agnostic, making it a pure operational utility rather than a product with its own customer-facing lifecycle.

---

## Decision Log

| Decision | Rationale |
|----------|-----------|
| Portfolio: Operations | Wasabi is a stateless AI processing service in service of Matcha's verification engine. It has no standalone customer value; it amplifies Matcha's throughput. Operations portfolio is the correct home. |
| ATLAS.md kept in-place | Source of truth during transition |
| FEATURE_*.md deferred | Requires engineering owner assignment and testable acceptance criteria |
| Stateless boundary enforced in PRODUCT.md | Wasabi stores nothing. Onigiri stores Phase 1 results; Matcha stores Phase 2 results. This boundary is critical to avoid distributed state management problems. |

---

## Links

- [PRODUCT.md](../PRODUCT.md)
- [PORTFOLIO.md](../../PORTFOLIO.md)
- [PORTFOLIO_CATALOG.md](../../../PORTFOLIO_CATALOG.md)
- [ATLAS.md](../ATLAS.md) (source)
