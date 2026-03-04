# Changelog: Atomic Architecture Migration

**Product**: DaVinci (Customer & Product Master Data)
**Portfolio**: Platform
**Changelog #**: 003
**Layer Affected**: Portfolio / Product / Capability
**Date**: 2026-03-04
**Author**: Claude (AI-assisted migration)

---

## What Changed

### Directory Restructure
- **Moved**: `product/davinci/` → `product/platform/davinci/`
- Git history fully preserved via `git mv` (all ATLAS.md, Architecture.md, changelogs, and artifacts/research/ tracked as renames)
- Working directory now lives under the `platform/` portfolio subdirectory in alignment with CLAUDE.md file system convention
- Note: `Architecture.md` casing preserved as-is (lowercase 'rchitecture') to avoid noise; separate cleanup decision if needed

### Files Created

| File | Layer | Description |
|------|-------|-------------|
| `product/platform/PORTFOLIO.md` | Portfolio | Platform portfolio definition: investment thesis (DaVinci as shared data infrastructure), known architecture gaps surfaced at portfolio level, portfolio-level OKRs, dependency note (all other portfolios depend on Platform) |
| `product/platform/davinci/PRODUCT.md` | Product | Full product spec: problem statement, value proposition, IS/IS NOT boundary, 7-capability registry, data flow architecture diagram (Mermaid), 7 consolidation resolution rules table, known architecture gaps (PDPA Consent Engine, Contact Compliance API contract, Collateral Master Data), KPIs |
| `product/platform/davinci/capabilities/golden-record/CAPABILITY.md` | Capability | Single deduplicated customer profile; davinci_customer_id as canonical key; identity core; KYC/AML status; product summary linkage (loans + policies — not full records); multi-subsidiary awareness |
| `product/platform/davinci/capabilities/consent-based-visibility/CAPABILITY.md` | Capability | PDPA consent registry; directed consent model (A→B ≠ B→A); default restricted visibility; API-level filtering by subsidiary + consent; immediate revocation enforcement; consent audit trail |
| `product/platform/davinci/capabilities/event-driven-synchronization/CAPABILITY.md` | Capability | Idempotent event consumption from Core Banking / Policy Admin / KYC / Onigiri; event schema registry with validation; DLQ for failed events; downstream Query API (profile + product summary + contact history) |
| `product/platform/davinci/capabilities/customer-data-change-management/CAPABILITY.md` | Capability | Change Request workflow (not direct edits); 4 change types: UPDATE_CONTACT / UPDATE_ADDRESS / UPDATE_IDENTITY / MERGE_DUPLICATES; auto-approve low-risk / reviewer-approve high-risk; duplicate detection; CustomerProfileChanged event propagation |
| `product/platform/davinci/capabilities/collection-contact-compliance/CAPABILITY.md` | Capability | Unified contact log across all products and subsidiaries; frequency check API (pre-contact query); block signal + ContactLimitReached event; ContactLimitApproaching warning; cross-product coordination. **BOUNDARY: Sensei is a consumer of these events — DaVinci is the sole owner of the contact log and frequency limits.** |
| `product/platform/davinci/capabilities/data-consolidation-engine/CAPABILITY.md` | Capability | Field-level authority matrix; 7 deterministic resolution rules; no-data-loss: Raw Event Store + FieldHistory + AlternativeValue; immutable Consolidation Log per decision; admin override with mandatory logging; self-check: post-write hash verification + nightly batch; Data Quality Report; Admin Page |
| `product/platform/davinci/capabilities/data-resolution-workflow/CAPABILITY.md` | Capability | ResolutionRequest entity (DaVinci-owned lifecycle); 3-tier classification: T1 CO Handle (same day) / T2 Needs Approval (3 days) / T3 Needs Verification via Matcha (5 days); entry channels: system conflict / customer call / branch walk-in / data quality alert. **BOUNDARY: DaVinci owns lifecycle state; Sensei creates/completes tasks only; Matcha owns T3 verification result.** |
| `product/platform/davinci/artifacts/diagrams/.gitkeep` | Infrastructure | Scaffold: artifacts/diagrams/ directory (was missing; research/ already existed with content) |

### Files Kept (Unchanged)
- `product/platform/davinci/ATLAS.md` — primary source of truth
- `product/platform/davinci/Architecture.md` — technical specifications (casing preserved)
- `product/platform/davinci/changelogs/CHANGELOG_001_initial_product_definition.md` — preserved
- `product/platform/davinci/changelogs/CHANGELOG_002_data_consolidation_engine.md` — preserved
- `product/platform/davinci/artifacts/research/atlas_vs_architecture_gap_analysis.md` — preserved
- `product/platform/davinci/artifacts/research/consolidation_engine_exploration.md` — preserved

---

## Known Architecture Gaps (Surfaced During Migration)

Three gaps identified in `artifacts/research/atlas_vs_architecture_gap_analysis.md` that are not yet addressed in ATLAS.md or Architecture.md:

| Gap | Description | Impact |
|-----|-------------|--------|
| PDPA Consent Engine | Consent registry + API-level visibility filtering not yet fully specified in Architecture.md | Capability 2 (consent-based-visibility) has business rules but no API contract |
| Contact Compliance API contract | Frequency Check API signature, response format, and event schema for ContactLimitReached not yet specified | Sensei cannot implement contact-compliance capability without this contract |
| Collateral Master Data | Onigiri captures collateral during origination; cross-product visibility of collateral requires a DaVinci capability that does not yet exist in ATLAS | Gap: Collateral Master Data capability not defined |

These gaps are flagged in `PRODUCT.md` under "KNOWN GAPS" and will require future @CAPABILITY and @PRODUCT sessions to resolve.

---

## Rationale

DaVinci is the shared data infrastructure layer for the entire enterprise — every other product (Onigiri, Matcha, Wasabi, Sensei) either sends events to or queries DaVinci. This cross-portfolio dependency makes DaVinci a Platform-level asset, not a Credit or Operations product. The `platform/` portfolio correctly reflects this: DaVinci is infrastructure, not a business-facing product.

---

## Decision Log

| Decision | Rationale |
|----------|-----------|
| Portfolio: Platform | DaVinci is consumed by every other portfolio. It is shared infrastructure, not a business-facing product. Platform portfolio correctly signals this investment category. |
| Known gaps surfaced in PRODUCT.md | Gap analysis results are too important to leave only in artifacts/research/. They are promoted to PRODUCT.md "KNOWN GAPS" so they are visible in the primary product reference document. |
| Contact compliance boundary triple-documented | The DaVinci-owns/Sensei-consumes boundary is documented in DaVinci PRODUCT.md, DaVinci collection-contact-compliance CAPABILITY.md, Sensei PRODUCT.md, and Sensei contact-compliance CAPABILITY.md. This boundary is frequently misunderstood and the redundancy is intentional. |
| ResolutionRequest ownership documented at both PRODUCT and CAPABILITY layers | DaVinci owns lifecycle state; Sensei executes tasks only. Documented in DaVinci PRODUCT.md and data-resolution-workflow CAPABILITY.md, and cross-referenced in Sensei PRODUCT.md. |
| Architecture.md casing not normalized | `Architecture.md` (lowercase 'rchitecture') preserved as-is to avoid noise in this migration commit. Separate cleanup decision if needed. |
| FEATURE_*.md deferred | Requires engineering owner assignment and testable acceptance criteria |

---

## Links

- [PRODUCT.md](../PRODUCT.md)
- [PORTFOLIO.md](../../PORTFOLIO.md)
- [PORTFOLIO_CATALOG.md](../../../PORTFOLIO_CATALOG.md)
- [ATLAS.md](../ATLAS.md) (source)
- [Architecture.md](../Architecture.md) (source)
- [Gap Analysis](../artifacts/research/atlas_vs_architecture_gap_analysis.md) (source)
