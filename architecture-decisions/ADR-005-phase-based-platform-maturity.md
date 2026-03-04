# ADR-005: Phase-Based Platform Maturity Model

**Status:** Accepted  
**Date:** 2025-12-01  
**Decider:** Salwan Mohamed  
**Domain:** Strategy / Governance  
**Project:** Nexus Platform Engineering Core

---

## Context

Organizations building internal platforms fail in a predictable pattern: they try to build everything at once, get overwhelmed, deliver nothing valuable, and abandon the effort. Alternatively, they deliver Phase 1 successfully and then face pressure to move to Phase 2 without consolidating the first phase's operations.

The question is: how do you structure platform evolution so that each phase delivers standalone value, advancement is earned not assumed, and the organization is never left in an unstable intermediate state?

---

## Constraints

- University has limited technical maturity — a complex platform delivered too fast won't be adopted
- Small platform team — can't maintain a full enterprise IDP while still building foundational infrastructure
- Budget approval cycles require demonstrable ROI at each phase
- Governance requirement: each phase must have observable, measurable exit criteria

---

## Decision

**Structure the platform as four distinct product phases (A/B/C/D), each with a standalone value proposition, explicit exit criteria, and no automatic advancement — advancement to the next phase requires explicit authorization.**

| Phase | Product | Tagline |
|-------|---------|--------|
| A | Infrastructure Control as a Service | *"Your infrastructure, under control, finally."* |
| B | Explainable Infrastructure as a Service | *"Know why before you ask."* |
| C | Platform Knowledge as a Service | *"Your platform remembers so you don't have to."* |
| D | Governed Self-Service Platform | *"Freedom within guardrails."* |

---

## Alternatives Considered

| Option | Reason Rejected |
|--------|-----------------|
| Big-bang platform delivery | Too high risk; nothing deliverable until everything is done; team burnout |
| Continuous delivery with no phase gates | No natural consolidation points; platform never stabilizes; always "in progress" |
| Capability-by-capability delivery (no phases) | Individual capabilities delivered without a coherent platform story; adoption fragmented |
| Maturity model as aspiration only | Without exit criteria, phases are cosmetic; teams advance without evidence |

---

## Trade-offs

**Given up:**
- Speed to full platform capability — phase gates slow the delivery of advanced features
- Flexibility to work on any capability in any order

**Gained:**
- Each phase is a complete, usable product — value delivered before perfection achieved
- Phase A can run indefinitely without Phase B ("advancement is optional, not assumed")
- Exit criteria make advancement a governance event, not a calendar event
- Stakeholders understand the roadmap because each phase has a named value proposition

---

## Consequences

**Current State at Sinai University:**
- Phase A: 🟢 Active — Infrastructure Control operational
- Phase B: 🟡 Partial — Explainable Platform in progress
- Phase C: ⚪ Planned
- Phase D: ⚪ Future

**The Platform Operating System** implements the phase gate mechanism: no phase auto-advances, all transitions require explicit Human Owner authorization, exit criteria are signal-validated (not opinion-validated).

---

## Evidence

- [nexus-platform-engineering](https://github.com/Salwan-Mohamed/nexus-platform-engineering) — Full phase definitions in docs/03-products/
- [platform-operating-system](https://github.com/Salwan-Mohamed/platform-operating-system) — Phase gate enforcement mechanism
- Core ADR-0114: Phase-Oriented Product Model
- Core ADR-0134: Phase Selection Assessment

---

## Operational Lesson

The most important implication of "advancement is optional" is that it reframes failure. If an organization is in Phase A and can't advance to Phase B, that's not failure — Phase A is a complete, functioning product. This reframe is critical for university stakeholder communication: the platform has value today, not only when it reaches Phase D.

---

*Reference Core ADR: nexus-platform-engineering/docs/01-decisions/adr/0114-phase-oriented-product-model.md*
