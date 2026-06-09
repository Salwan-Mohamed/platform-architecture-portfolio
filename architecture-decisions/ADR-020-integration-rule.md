# ADR-020: Integration Rule

**Status:** Accepted  
**Date:** 2026-06-09  
**Source:** ARCH-0004 (Integration Rule, 2026-05)  
**Project:** Sinai University Platform — Platform Governance Layer

---

## Context

The Sinai University Platform has 16+ discrete components: RKE2, ArgoCD, Keycloak, Cilium, Prometheus, Grafana, Loki, Robusta, Hubble, Tekton, Terraform, AWX, Sealed Secrets, CAPV, Longhorn, Nginx Ingress. Each component has integration points with other components.

Without an explicit integration rule, every component can integrate with every other component in whatever way is operationally convenient. The result is an integration mesh where the dependencies are implicit, the contracts are undocumented, and changing any component requires auditing all the implicit dependencies it may have created.

ARCH-0004 defines the Integration Rule: what constitutes a permitted integration, what constitutes a forbidden coupling, and how the rule is enforced.

## Constraints

- GitOps authority model applies — integration is declared in Git, not wired ad-hoc at runtime
- The Core/IA dependency rule from ADR-018 must be enforced at the integration level — IA components must not become Core Platform runtime dependencies
- A 4-person team cannot audit a full integration mesh for every change — the rule must be simple enough to apply at design time, not only at review time
- Identity integration (everything authenticates via Keycloak OIDC) is an existing decision (ADR-009) — the integration rule must be consistent with it

## Decision

**Platform components integrate via capability contracts and published interfaces. Direct component-to-component coupling is forbidden unless the dependency is part of the component's defined capability contract. Integration direction follows the platform authority hierarchy: Core Platform components do not depend on Capability Extensions; Capability Extensions may depend on Core Platform interfaces.**

A **permitted integration** is:
- A component consuming a published interface of another component (e.g., Grafana reading from Prometheus via the Prometheus HTTP API)
- A domain registering a capability contract that the platform fulfils (per ADR-019)
- An Infrastructure Automation component writing to infrastructure via a defined execution interface (Terraform vSphere provider, Ansible modules), with Core Platform components not depending on that write path
- An observability component (Prometheus, Loki, Hubble) collecting signals from other components via their published metrics/log endpoints

A **forbidden coupling** is:
- A Core Platform component whose runtime operation requires an Infrastructure Automation component to be available (violates ADR-018 Core/IA dependency rule)
- A component that reads or writes another component's internal state directly (bypassing the published interface)
- A circular dependency between two components at the same layer
- A domain application coupling directly to a platform component's internal configuration (domain must use the contract model per ADR-019)

The **integration authority hierarchy:**
- Layer 0: Infrastructure (compute, network, storage) — no platform dependencies
- Layer 1: Core Platform (RKE2, Cilium, Keycloak, ArgoCD, Sealed Secrets) — no IA dependencies; depends only on Layer 0
- Layer 2: Platform Services (Prometheus, Grafana, Loki, Robusta, Hubble, Tekton, Longhorn) — depends on Layer 1; no circular dependencies within Layer 2
- Layer 3: Infrastructure Automation (Terraform, AWX, Ansible) — extends Layer 0; must not be a Layer 1 runtime dependency
- Layer 4: Application Domains — depends on Layer 1 and Layer 2 interfaces only; uses contract model per ADR-019

Integration declarations live in the GitOps repository as dependency manifests. Any integration not declared in the manifests is an integration violation.

## Alternatives Considered

**Ad-hoc integration** — components integrate as needed, dependencies documented post-hoc. Rejected: this is the status quo in many platform teams. The documentation never catches up with the implementation. When a component needs to change, nobody knows all the implicit dependencies.

**Strict layering with no cross-layer dependencies** — each layer can only depend on the layer immediately below it. Rejected: over-rigid. Grafana legitimately integrates with both Prometheus (Layer 2) and Loki (Layer 2) — both are valid Layer 2 integrations. Forcing strict adjacency-only dependencies would require artificial intermediate layers.

**Runtime enforcement via service mesh** — use Istio or Linkerd to enforce integration policy at runtime. Rejected: ADR-010 explicitly excluded service mesh (Cilium L7 policy is sufficient). Runtime enforcement is also reactive — it catches violations after deployment, not at design time. Design-time rule application is more valuable.

## Trade-offs

**Accepted:** Integration declarations in Git require discipline. Engineers must declare integrations before implementing them. This adds a design step. The payoff is that integration dependencies are auditable and the integration rule can be reviewed at PR time, not discovered during an incident.

**Accepted:** Some legitimate integrations may require rule review. The integration rule is a model, not an exhaustive list. Novel integration patterns may require a review against the rule before implementation. For a 4-person team, that review is a brief conversation, not a bureaucratic process.

**Gained:** Every component change can be assessed for integration impact by reading the declared dependency manifests. There are no hidden dependencies. Changing the Prometheus HTTP API response format affects only the components whose integration manifests declare a dependency on it.

## Consequences

- All platform component integrations are declared in the GitOps repository before implementation
- New integrations are reviewed against the authority hierarchy at PR time
- The Core/IA dependency rule (ADR-018) is enforced at the integration declaration level — any proposed integration that makes a Layer 1 component depend on Layer 3 is rejected at PR review
- Integration violations discovered during incident response are treated as governance events requiring a post-incident ADR update
- This framework enables the platform to reason about blast radius for any component change: "which components have declared a dependency on X?"

## Evidence

- ARCH-0004 approved 2026-05 with Human Owner Decision Payload verbatim
- ADR-018 (IA Positioning) establishes the Core/IA dependency rule that this ADR enforces at the integration level
- ADR-019 (Domain Application Framework) establishes the contract model that this ADR's Layer 4 integration rule references

## Principal-Level Signal

Integration debt compounds faster than any other kind of technical debt. Each implicit dependency is a future constraint on every component involved. The Integration Rule is not about preventing engineers from connecting components — it is about ensuring those connections are declared, reviewed, and consistent with the architecture's authority hierarchy. A platform where every integration is explicit is a platform where every change is predictable. That predictability is the difference between operating infrastructure and managing entropy.
