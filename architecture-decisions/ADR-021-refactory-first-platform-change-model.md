# ADR-021: Refactory-First Platform Change Model

**Status:** Accepted  
**Date:** 2026-06-09  
**Source:** ARCH-0005 (Refactory-First Platform Change Model, 2026-05)  
**Project:** Sinai University Platform — Platform Governance Layer

---

## Context

The Sinai University Platform has matured through Phase A and into Phase B with a disciplined tool selection: specific tools were chosen, alternatives were explicitly rejected, and ADRs document the reasoning. Despite this discipline, there is a recurring pressure pattern: a new operational need arises, and the reflex is to add a new tool or a new component rather than assess whether the existing model can be extended or refactored to address the need.

This pressure is natural. New tools are well-documented. The existing model requires understanding. Adding is faster than refactoring. But over time, the result is a platform that has 30 components where 15 would have been sufficient — each addition narrowing the team's operational bandwidth further.

ARCH-0005 defines the Refactory-First Platform Change Model: any platform change must first demonstrate that the existing model is insufficient before adding new complexity. The burden of proof is on addition, not on conservation.

## Constraints

- 4-person team operating 3-site infrastructure cannot sustain unbounded component growth
- Each additional tool requires: learning, monitoring, upgrading, troubleshooting, backup — operational cost that scales with the team, not with the infrastructure
- The ARCH-series ADRs (0001–0004) have already invested in governance frameworks — new changes must be assessed against existing frameworks before introducing new ones
- The Phase C execution plan has explicit deferrals (AWX, OpenTelemetry, distributed tracing, service mesh) — those deferrals are not just delays, they are architectural commitments to evaluate before adding

## Decision

**Before any net-new platform component or capability extension is added, the platform change proposal must demonstrate that the existing model is insufficient to address the need. Refactoring or extending an existing component is the default path. Addition is the exception, not the default.**

A **platform change proposal** must answer, in order:
1. What existing capability or component was evaluated for this need?
2. Why is refactoring or extending the existing model insufficient?
3. If addition is required: what is the operational cost (learning, monitoring, upgrade, troubleshooting) and does the team have capacity?
4. What is the exit path if the new component proves insufficient — can it be removed without cascading impact?
5. Does the addition comply with the Integration Rule (ADR-020) and the Core/IA dependency rule (ADR-018)?

This is not a veto on addition — it is a structured forcing function. If the existing model genuinely cannot address the need, addition is the correct answer. The discipline is ensuring that assessment happens before the addition, not after.

**Specific deferred components** named in ADR-016 and the Phase C DEFERRALS.md:
- **AWX**: not because it lacks value, but because Ansible CLI is sufficient for initial deployment. AWX activates when centralised execution, credential vault, audit logging, and RBAC for automation are operationally required — not when they are theoretically useful.
- **OpenTelemetry/distributed tracing**: not because tracing lacks value, but because Prometheus + Loki + Hubble covers the observability needs of the current platform services. Tracing activates when a service's failure modes require distributed trace analysis that metrics and logs cannot provide.
- **Service mesh (Istio/Linkerd)**: explicitly excluded. Cilium L7 network policy covers the requirement. A service mesh would add a sidecar injection model, a separate control plane, and a separate policy language for marginal additional capability at current scale.

These deferrals are reviewed at each phase gate. The review question is: has the evidence changed? If the existing model is now demonstrably insufficient, the deferral is lifted and the addition proceeds through the standard ADR process.

## Alternatives Considered

**Addition-first model** — add tools when they provide value, remove them if they prove unnecessary. Rejected: removal is harder than addition. Tools accumulate integrations (per ADR-020), operational habits, and documentation. The cost of addition is front-loaded (learning, deployment); the cost of an unnecessary tool is continuous (operational overhead, upgrade cadence, monitoring surface). The asymmetry favours conservatism.

**No addition policy** — platform is frozen at its current component set. Rejected: too rigid. Operational needs evolve. The platform at Phase D will require capabilities the platform at Phase A did not. The question is not whether to add, but when and why.

**Case-by-case discretion without a framework** — each addition assessed informally. Rejected: informal assessment does not produce audit-trail evidence. The platform's governance framework requires decisions to be documented. The Refactory-First model provides the documentation structure for addition decisions.

## Trade-offs

**Accepted:** Some additions will take longer because the refactoring assessment takes time. An engineer who has identified a useful tool will need to document why the existing model is insufficient before proceeding. That documentation is the trade-off.

**Accepted:** Some legitimate operational improvements will be deferred. The assessment process may conclude that the existing model is insufficient but the team lacks operational capacity to absorb the addition right now. The addition is then queued, not rejected permanently.

**Gained:** Platform complexity grows only when justified. Every addition has a documented rationale that can be reviewed at the next phase gate. Deferred components have documented review criteria. The team's operational bandwidth is protected by default, not by accident.

## Consequences

- All future platform change proposals include a Refactory-First assessment as the first section of the proposal ADR
- Deferred components (AWX, OpenTelemetry, service mesh) are reviewed at each phase gate against their deferral criteria
- The Integration Rule (ADR-020) and Core/IA dependency rule (ADR-018) are applied to every addition as part of the assessment
- Platform component count is a tracked metric — growth requires justification
- This model applies to governance frameworks as well as tools: before adding a new governance process, assess whether the existing framework can be extended

## Evidence

- ARCH-0005 approved 2026-05 with Human Owner Decision Payload verbatim
- Phase C DEFERRALS.md documents the current deferral set with explicit review criteria
- The platform's ADR library (ADR-001 to ADR-021) demonstrates the Refactory-First model in practice: every tool selection includes an explicit rejection of alternatives with reasoning

## Principal-Level Signal

The most expensive platform decision is not a bad tool choice — it is a good tool choice made before the team was ready to operate it. The Refactory-First model shifts the question from "should we use this tool?" to "does the evidence show we need this tool now?" That shift matters because the answer to the first question is almost always "yes, eventually." The answer to the second question determines whether the addition serves the platform or burdens the team. A Principal Engineer's job is not to collect the best tools. It is to build the minimum platform that consistently delivers on its capability promises — and to know the difference between the two.
