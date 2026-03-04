# Changelog: Atomic Architecture Migration

**Product**: Onigiri (Loan Origination System)
**Portfolio**: Credit
**Changelog #**: 002
**Layer Affected**: Portfolio / Product / Capability
**Date**: 2026-03-04
**Author**: Claude (AI-assisted migration)

---

## What Changed

### Directory Restructure
- **Moved**: `product/onigiri/` → `product/credit/onigiri/`
- Git history fully preserved via `git mv` (all ATLAS.md, ARCHITECTURE.md, and previous changelog files tracked as renames)
- Working directory now lives under the `credit/` portfolio subdirectory in alignment with CLAUDE.md file system convention

### Files Created

| File | Layer | Description |
|------|-------|-------------|
| `product/credit/PORTFOLIO.md` | Portfolio | Credit portfolio definition: investment thesis (Onigiri + future Loan Servicing + Collections), portfolio-level OKRs, cross-product dependencies on DaVinci, strategic roadmap |
| `product/credit/onigiri/PRODUCT.md` | Product | Full product spec: problem statement, value proposition, IS/IS NOT boundary, 4-capability registry table, underwriting workflow state machine (Mermaid), integration map with RECEIVES/SENDS, product-level KPIs |
| `product/credit/onigiri/capabilities/smart-form/CAPABILITY.md` | Capability | Dynamic form builder: product-driven field rules, document checklist engine, session persistence, application state machine |
| `product/credit/onigiri/capabilities/underwriting-workflow/CAPABILITY.md` | Capability | 4-phase underwriting: Preparation → Underwriting → Credit Approval → Document Preparation; compliance-locked steps; CO/BOMA/AVP role gating |
| `product/credit/onigiri/capabilities/loan-campaign-configuration/CAPABILITY.md` | Capability | Campaign config: eligibility rules, pricing bands, document requirements, campaign lifecycle (DRAFT → ACTIVE → ARCHIVED) |
| `product/credit/onigiri/capabilities/risk-assessment-engine/CAPABILITY.md` | Capability | JMESPath-based rule evaluation engine: Strategy → Policy → Rule hierarchy; configurable scoring; explanation output |
| `product/credit/onigiri/artifacts/diagrams/.gitkeep` | Infrastructure | Scaffold: artifacts/diagrams/ directory (was missing) |
| `product/credit/onigiri/artifacts/research/.gitkeep` | Infrastructure | Scaffold: artifacts/research/ directory (was missing) |
| `product/PORTFOLIO_CATALOG.md` | Portfolio | Master registry replacing PRODUCT_CATALOG.md; groups all products by portfolio with status and links |

### Files Kept (Unchanged)
- `product/credit/onigiri/ATLAS.md` — primary source of truth; PRODUCT.md and CAPABILITY.md are extractions, not replacements
- `product/credit/onigiri/changelogs/CHANGELOG_001_initial-atlas.md` — preserved as-is

---

## Rationale

CLAUDE.md defines a 4-layer Atomic Architecture (Portfolio → Product → Capability → Feature). The previous flat structure (`product/onigiri/ATLAS.md`) did not expose the portfolio layer or the capability layer as navigable, standalone documents. This migration:
1. Makes every layer independently readable and linkable
2. Establishes the `credit/` portfolio as the home for the full lending lifecycle (Onigiri now; Loan Servicing and Collections future)
3. Creates CAPABILITY.md stubs that will be progressively filled during @CAPABILITY sessions as engineering owners are assigned

---

## Decision Log

| Decision | Rationale |
|----------|-----------|
| ATLAS.md kept in-place | Remains as source of truth during transition; PRODUCT.md and CAPABILITY.md derive from it, not replace it |
| FEATURE_*.md deferred | Features require engineering owner assignment and testable acceptance criteria — deferred to future @FEATURE sessions |
| Portfolio: Credit | Onigiri covers the origination lifecycle; future Loan Servicing and Collections will be separate products in the same Credit portfolio |
| `git mv` used, not OS `mv` | Preserves full git history for all files; prevents false "file deleted + file created" in git log |

---

## Links

- [PRODUCT.md](../PRODUCT.md)
- [PORTFOLIO.md](../../PORTFOLIO.md)
- [PORTFOLIO_CATALOG.md](../../../PORTFOLIO_CATALOG.md)
- [ATLAS.md](../ATLAS.md) (source)
