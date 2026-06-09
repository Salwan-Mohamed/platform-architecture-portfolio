# Discovery Report: ARCH-Series ADRs (ARCH-0001 to ARCH-0005)

> **Agent:** Discovery Agent  
> **Date:** 2026-06-09  
> **Source:** nexus-platform-product-sinai-university/docs/adr/ARCH-000X  
> **Status:** NEW TRACK — 5 ADRs not previously captured in portfolio

---

## What This Track Is

The `ARCH-` series is a new ADR track introduced in May 2026, distinct from the `IMPL-` and `impl-` series. Where IMPL ADRs document tool/product decisions ("we use Keycloak", "we use Cilium"), ARCH ADRs document architectural governance models — the structural principles the platform enforces, independent of specific tools.

All five are **ADVISORY DRAFT** status — authored by Claude Code (Operator), with Human Owner Decision Payload recorded verbatim. They represent a governance maturation event: the platform formalising its own operating model as architecture decisions.

---

## ARCH ADR Inventory

| ADR | Title | Date | Size | Core Insight |
|-----|-------|------|------|-------------|
| ARCH-0001 | Capability Maturity Model | 2026-05-07 | 24KB | 7-level progression (disabled → observed → backup → read-only-inventory → safe-write → controlled-write → full-managed); gates write-capability to maturity level |
| ARCH-0002 | Infrastructure Automation Positioning | 2026-05-08 | 33KB | Core platform must NOT depend on IA; failure-mode commitments per component; visibility-first before write-capability; modes bound to ARCH-0001 levels |
| ARCH-0003 | Domain Application Framework | 2026-05-?? | 41KB | Domain plugin model; how application capabilities are added to the platform without coupling to the core |
| ARCH-0004 | Integration Rule | 2026-05-?? | 43KB | How platform components integrate — what is a permitted integration vs. a forbidden coupling |
| ARCH-0005 | Refactory-First Platform Change Model | 2026-05-?? | 15KB | Platform change must refactor before adding; no net new complexity without evidence the existing model is insufficient |

---

## Key Architectural Signals

### ARCH-0001: Capability Maturity Model
- 7-level progression is canonical for every capability in the platform
- Level 1 (disabled) → Level 4 (read-only-inventory) = no write risk
- Level 5 (safe-write) → Level 7 (full-managed) = write-capable; all require Human Owner approval
- Exit criteria: evidence pack + validation + rollback path + risk understanding + owner review — **not calendar-based**
- Cadence: event-driven at evidence milestones, wave gates, onboarding, write-capable transitions, quarterly minimum
- This directly extends ADR-012 (evidence-gated phase transitions) to capability-level granularity

### ARCH-0002: Infrastructure Automation Positioning  
- Core/IA dependency rule with **named failure-mode commitments**: RKE2, ArgoCD, Keycloak, Cilium, Observability must all survive IA being disabled/degraded/removed
- Visibility-first is **per-domain**, not a global blocker — low-risk functions (DNS/DHCP) can enter safe-write before high-risk domains (SAN zoning, firewall policy)
- High-risk domains require controlled-write maturity + explicit Human Owner approval + rollback validation — not just safe-write
- AWX explicitly deferred (IMPL-0030 §2.4) — Terraform + Ansible are the current execution stack
- This enriches ADR-016 (Infrastructure Execution Stack) with the governance positioning layer

### ARCH-0003: Domain Application Framework
- Defines how capabilities (application domains) plug into the platform without coupling to core
- Domain plugin pattern: capability registers via contract, platform routes; no direct component coupling
- Enables the platform to grow without the core becoming a monolith

### ARCH-0004: Integration Rule
- Defines what constitutes a permitted integration (contract-based, loosely coupled) vs. a coupling violation
- Prevents "integration creep" where every component starts depending on every other component

### ARCH-0005: Refactory-First
- Any platform change must first demonstrate the existing model is insufficient before adding new complexity
- Anti-pattern: adding new tools when existing tools could be extended or refactored
- Forces architectural conservatism — complexity must be justified, not assumed

---

## Portfolio ADRs to Generate

| Portfolio ADR | Source | Topic |
|---------------|--------|-------|
| ADR-017 | ARCH-0001 | Capability Maturity Model — 7-level progression |
| ADR-018 | ARCH-0002 | Infrastructure Automation Positioning — Core/IA independence |
| ADR-019 | ARCH-0003 | Domain Application Framework |
| ADR-020 | ARCH-0004 | Integration Rule |
| ADR-021 | ARCH-0005 | Refactory-First Platform Change Model |

---

## Still Pending from Previous Sync

| Portfolio ADR | Source | Topic | Status |
|---------------|--------|-------|--------|
| ADR-017* | impl-0019, impl-0020 | Infrastructure TDD + Static Analysis | ⏳ Superseded by ARCH numbering above — renumber to ADR-022 |
| ADR-018* | IMPL-0021, IMPL-0022, IMPL-0031 | DNS + Storage + North-South Exposure | ⏳ Superseded — renumber to ADR-023 |
