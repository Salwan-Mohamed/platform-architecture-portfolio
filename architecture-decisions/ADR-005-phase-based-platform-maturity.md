# ADR-005: Phase-Based Platform Maturity Model

**Status:** Accepted  
**Date:** 2025-12-20  
**Decider:** Salwan Mohamed  
**Project:** Nexus Platform Engineering — Strategic Model  

---

## Context

Platform engineering projects fail in two characteristic ways: they either under-deliver (never moving beyond basic infrastructure) or over-engineer (building self-service portals before the underlying infrastructure is reliable). Both failure modes stem from the same root cause: **no explicit model for what "done" means at each stage of platform maturity.**

At Sinai University, the initial temptation was to immediately build a developer portal (Phase D capability) before the infrastructure was even under version control (Phase A requirement). Without a maturity model, every conversation about the platform devolved into a debate about future features rather than a clear plan for the current state.

---

## Constraints

| Constraint | Impact |
|------------|--------|
| University leadership wants "modern platform" immediately | Risk of building showroom features on unstable foundation |
| Ops team is not yet fully GitOps-proficient | Phase D self-service would be unsafe before Phase A is solid |
| Budget is phased — not a single large investment | Natural alignment between budget cycles and platform phases |
| Platform must remain operable if advanced features are removed | Each phase must be independently stable |
| Need to justify investment at each governance checkpoint | Phases provide natural evidence points for leadership reporting |

---

## Decision

**Adopt a four-phase platform maturity model (A→B→C→D) where each phase is a stable, independently valuable product. Advancement to the next phase is optional, not assumed.**

| Phase | Product | What It Delivers |
|-------|---------|------------------|
| **A** | Infrastructure Control as a Service | Your infrastructure, under control, finally |
| **B** | Explainable Infrastructure as a Service | Know why before you ask |
| **C** | Platform Knowledge as a Service | Your platform remembers so you don't have to |
| **D** | Governed Self-Service Platform | Freedom within guardrails |

Sinai University is currently in **Phase A (Active) / Phase B (Partial)**.

---

## Alternatives Considered

| Option | Reason Rejected |
|--------|----------------|
| **Build everything at once** | No stable checkpoint model; failure at any layer fails the whole platform; politically dangerous when budget is phased |
| **"Crawl/Walk/Run" model** | Too vague; doesn't define what capabilities exist at each stage or what evidence proves readiness to advance |
| **Maturity Model Capability Assessment (CMMI-style)** | Scores maturity but doesn't produce a roadmap or a product; doesn't answer "what do we build next?" |
| **No explicit model — organic growth** | Known failure mode: platform accumulates technical debt without direction; no way to say no to premature requests |

---

## Trade-offs

**What we gained:**
- Every phase produces a stable, independently valuable platform state — no wasted work if the organization pauses
- Clear vocabulary for leadership: "We are completing Phase A before starting Phase B" is a defensible position
- Phase gates provide natural evidence points for governance reporting
- The model prevents "shiny feature" scope creep: features are gated by phase readiness
- Advancement is optional: an organization can be permanently successful at Phase A

**What we gave up:**
- The model creates a perception of slower delivery — "why can't we have self-service now?"
- Phase definitions require upfront design thinking before any infrastructure is built
- Teams accustomed to "feature roadmaps" must reframe around "capability phases"

---

## Consequences

- Nexus Platform Core ADRs are organized by phase requirement — each ADR declares its minimum phase
- Sinai University Platform roadmap is structured as Phase A completion → Phase B gap analysis
- University leadership receives quarterly phase progress reports with quantified capability coverage
- Platform Operating System Phase model aligns with this ADR (Phase 10 = observability/baseline establishment before decision enforcement)

---

## Evidence

- [nexus-platform-engineering Phase Products](https://github.com/Salwan-Mohamed/nexus-platform-engineering/tree/main/docs/03-products) — Phase A/B/C/D product definitions
- [platform-operating-system](https://github.com/Salwan-Mohamed/platform-operating-system) — Operational phase model in practice
- Core ADR-0114: Phase-Oriented Product Model
- Core ADR-0134: Phase Selection Assessment

---

## Principal-Level Signal

> The phase model is not about slowing down — it is about delivering value that compounds. A Phase A platform that is 100% reliable is worth more than a Phase D platform that is 40% reliable. The question to ask at every roadmap meeting is: **"Are we advancing phases or patching foundations?"** The answer should always be foundations first.
