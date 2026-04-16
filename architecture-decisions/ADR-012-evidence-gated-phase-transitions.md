# ADR-012: Evidence-Gated Phase Transitions

**Status:** Accepted  
**Date:** 2026-04-16  
**Source:** IMPL-0027 (Phase Transition Criteria)  
**Project:** Sinai University Platform

---

## Context

The Nexus Platform uses a four-phase maturity model (Phase A: Infrastructure Control → Phase B: Explainable Platform → Phase C: Platform Knowledge → Phase D: Governed Self-Service). Phase advancement is optional and intentional — not automatic with time.

Without formal advancement criteria, phases become cosmetic labels. A team can declare "Phase B" because the calendar says so while running a Phase A-equivalent operation. Stakeholder pressure ("when will we be in Phase C?") can drive premature advancement. The result is phases that describe aspiration, not reality.

IMPL-0027 defines the Sinai University implementation of phase transition governance: explicit evidence gates that must be satisfied before advancement is permitted. Leadership declares readiness. Engineering provides evidence. Both must agree.

## Constraints

- University governance board requires audit-trail of platform capability claims
- Small team cannot sustain a phase that was advanced before the team was ready
- Platform phases were designed as reliability checkpoints, not milestone celebrations
- ADR-005 (phase-based platform maturity) established the model; this ADR implements the gate mechanism

## Decision

**Phase transitions require documented evidence satisfying the gate criteria for the current phase. Gate criteria are defined in advance, not at the time of the transition. Advancement is approved by the Human Owner against evidence provided by the engineering team.**

Each phase has a written set of gate criteria — observable, testable conditions that confirm the phase's intent is met. Examples: Phase A gate requires 100% of infrastructure changes committed to Git before execution. Phase B gate requires SLOs defined for all Tier-1 services with 30 days of baseline data.

The transition process: engineering team documents evidence against each gate criterion → Human Owner reviews → advancement is recorded as a commit to the platform governance repository. Calendar pressure does not override gate criteria.

Phases are non-reversible in direction but are expected to be stable for months. "We are in Phase B" is a claim with evidentiary backing, not a project management label.

## Alternatives Considered

**Calendar-based phase advancement** — advance phases on a schedule (e.g., Phase B after 3 months). Rejected: schedule has no relationship to platform capability. A 3-month calendar date can be met while the platform is still in a Phase A state.

**Continuous advancement (no formal gates)** — treat phases as a continuum, advance informally. Rejected: removes the forcing function for governance maturity. Without gates, phases lose meaning and the model degrades to a marketing label.

**Engineering team self-certifies** — team declares advancement without Human Owner review. Rejected: creates a conflict of interest; the team has an incentive to advance. Evidence review requires a second perspective.

## Trade-offs

**Accepted:** Phase advancement takes longer than calendar-based approaches. The team cannot advance until they can produce evidence. For a small team with operational responsibilities, 30-day baseline requirements can feel slow. Mitigation: gate criteria are known in advance so the team can plan evidence collection.

**Accepted:** Documentation overhead per phase. Gate evidence must be written down, not just demonstrated. This takes time. The payoff: the governance board has an audit trail and the team has a reference for what "ready" meant when they advanced.

**Gained:** Platform capability claims are credible. When the portfolio states "Phase A complete," it means infrastructure is 100% under version control and GitOps is enforced — not that someone checked a box. That credibility matters for the Principal-level career narrative.

## Consequences

- Phase A gate criteria are retroactively documented and evidence is recorded (Phase A is complete)
- Phase B gate criteria are active — SLO baseline window started on Phase B entry
- Phase C gate criteria are documented but not yet approaching
- All phase gate evidence lives in the governance repository as committed markdown files
- No phase advancement occurs in this platform without a Human Owner approval commit

## Evidence

- IMPL-0027 approved 2026-02-04 by Human Owner
- ADR-005 (Phase-Based Platform Maturity) established the four-phase model — this ADR implements the enforcement mechanism
- Phase A declared complete with evidence: 100% infrastructure in Git, zero unauthorized changes since GitOps adoption

## Principal-Level Signal

Phase gates are where governance becomes real. Any model without enforcement is a suggestion. The evidence gate mechanism converts the phase maturity model from a framework you describe in job interviews into a practice you can prove. The distinction matters: a Principal Engineer does not just design governance models — they build the enforcement mechanisms that make governance non-optional.
