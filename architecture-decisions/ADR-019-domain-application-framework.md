# ADR-019: Domain Application Framework

**Status:** Accepted  
**Date:** 2026-06-09  
**Source:** ARCH-0003 (Domain Application Framework, 2026-05)  
**Project:** Sinai University Platform — Platform Governance Layer

---

## Context

The Sinai University Platform currently supports a known set of application capabilities: the LMS, Admission system, Telepresence/UC infrastructure, and internal platform services. Each of these has different network requirements, different identity needs, different storage profiles, and different operational characteristics.

Without an explicit framework for how application domains plug into the platform, each new application domain creates direct couplings into the platform core — its own firewall rules, its own DNS entries, its own Keycloak realm, its own storage class. Over time, the platform becomes a monolith where changing any component requires understanding all other components that coupled to it.

ARCH-0003 defines the Domain Application Framework: the pattern by which application capabilities register with the platform via capability contracts, without directly coupling to the platform's internal implementation.

## Constraints

- Platform core must remain modifiable without requiring changes to application domains
- Application domains must be onboardable by site ops engineers without Platform Architect involvement for every onboarding
- The framework must accommodate the heterogeneous application landscape: LMS (web app), Admission (web app), Telepresence (UC/voice infrastructure), and future domains
- GitOps authority model applies — domain registration is a Git operation, not a manual platform configuration

## Decision

**Application domains join the platform by registering capability contracts against platform-provided interfaces. No application domain couples directly to platform internal components. The platform routes and enforces; the domain declares what it needs.**

A domain capability contract declares:
- **Identity requirements:** which Keycloak realm, which OIDC client configuration, which AD groups map to which roles
- **Network requirements:** which services are exposed north-south (via Ingress/Gateway API), which east-west policies are required (via CiliumNetworkPolicy)
- **Storage requirements:** which storage class, what retention, what backup cadence
- **Observability requirements:** which metrics endpoints to scrape, which log namespaces to collect, which SLOs to register

The platform enforces these contracts through its existing capability implementations. A domain does not configure Keycloak directly — it declares its OIDC requirements; the platform provisions the Keycloak client. A domain does not write its own Cilium network policies — it declares its network contract; the platform applies the policies.

Domain onboarding is a GitOps operation: the domain contract is committed to the GitOps repository, ArgoCD applies it, platform controllers reconcile the contract into the appropriate platform configurations.

## Alternatives Considered

**Direct coupling per domain** — each application domain manages its own Keycloak realm, its own network policies, its own storage provisioning. Rejected: this is the current informal state. It produces N copies of partially-consistent platform configuration, each maintained independently, each a coupling point that constrains future platform evolution.

**Platform team owns all domain configuration** — no self-service; all domain requirements go through the Platform Architect. Rejected: does not scale with the team size. With 1 Principal + 3 site ops and a growing application landscape, this bottleneck is operational debt.

**Namespace-based isolation only** — each domain gets a namespace, isolation is Kubernetes RBAC + namespace boundaries. Rejected: namespace isolation does not address the higher-order coupling problem at the identity, network, and observability layers. A domain still needs to know how to configure Keycloak, how to write CiliumNetworkPolicy, how to register SLOs.

## Trade-offs

**Accepted:** Contract design overhead. Defining a well-structured capability contract for a new domain type takes time upfront. The payoff: every subsequent domain of the same type follows the same contract pattern and is onboarded without Platform Architect involvement.

**Accepted:** Platform must implement the contract interface before domains can use it. The framework is only useful if the platform enforces the contracts. This means platform team investment before domain teams benefit.

**Gained:** Platform internals are changeable without coordinating with domain teams. If the platform migrates from Nginx Ingress to Cilium Gateway API, domain contracts declare "expose this service externally" — the platform fulfils the contract using whatever implementation is current. Domains are isolated from implementation changes.

## Consequences

- New application domain onboarding is a GitOps commit of a capability contract
- Platform controllers reconcile contracts into Keycloak clients, CiliumNetworkPolicies, StorageClass bindings, and Prometheus scrape configs
- Platform core components (Keycloak, Cilium, Longhorn, Prometheus) are not directly configured by domain teams
- Domain capability contracts are versioned in Git — the history of what each domain required is auditable
- This framework enables the platform to evolve its internal implementations (e.g., swap Longhorn for another storage provider) without requiring domain teams to change their contracts

## Evidence

- ARCH-0003 approved 2026-05 with Human Owner Decision Payload verbatim
- Phase B Sinai University application landscape: LMS2, Admission, Expressway/Telepresence (9 published services at Kantra-site per network architecture documentation)

## Principal-Level Signal

A platform that requires domain teams to understand its internals is not a platform — it is a managed hosting service. The Domain Application Framework is what makes the platform a platform: domain teams declare requirements, platform enforces them. The implementation details of how Keycloak is configured, how Cilium policies are applied, how Longhorn provisions volumes — these are platform concerns, not domain concerns. That separation is not a luxury. It is the condition under which the platform can evolve without breaking everything that runs on it.
