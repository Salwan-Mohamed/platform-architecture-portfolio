# Career Narrative — Salwan Mohamed
## Senior Platform Engineer → Principal Platform Engineer

---

## The Arc

Most engineers build systems. I build systems that govern systems.

The distinction matters. A Senior Platform Engineer makes the platform work. A Principal Platform Engineer makes the platform *trustworthy* — at scale, across organizational boundaries, over time, without the original author in the room.

This portfolio documents that transition: from operator to architect, from implementer to doctrine-maker, from solving individual problems to designing frameworks that solve families of problems.

---

## Chapter 1: Learning That Infrastructure Is a People Problem (2024–early 2025)

I started where most platform engineers start — fixing broken things. Kubernetes clusters that wouldn't join. Network automation that ran but wasn't trusted. VPN tunnels that were configured correctly on paper but wrong in production.

The technical problems were tractable. The real problems were organizational:
- **Nobody knew what "correct" looked like** — there was no single source of truth
- **Changes happened through tribal knowledge** — undocumented, untraceable, unrepeatable
- **Ownership was ambiguous** — when things broke, the question was always "whose problem is this?"

This is when I started thinking about infrastructure as a coordination problem, not a configuration problem.

**Key repos from this phase:**
- `network-automation` — First automation work, Aruba switches via AWX
- `ansible-role-k8s-ha-cluster` — HA Kubernetes via Ansible: learned that automation without governance creates technical debt faster than manual processes
- `kubernetes-ha-cluster` — Production HA cluster: learned that Day 2 operations are the real design constraint

---

## Chapter 2: Designing for Multi-Site Reality (2025)

Sinai University is not one site. It's three campuses — Kantra, Katamia, Arish — each with independent internet breakout, local FortiGate firewalls, and real users who need reliable connectivity between all of them.

The naive solution was hub-spoke VPN: all traffic through a central datacenter. I rejected it.

**The decision that defined this phase:** Full-mesh IPsec VPN between branch edge firewalls, with no hub dependency.

Why? Because hub-spoke creates a single point of failure in both connectivity and governance. If the datacenter is unreachable, branches can't talk to each other — even for traffic that never needed to touch the datacenter. Autonomy at the edge is not just a performance optimization; it's a resilience architecture.

This wasn't a networking decision. It was an organizational decision expressed as a network topology.

**What I built:**
- 3-site full-mesh FortiGate VPN architecture
- 67 firewall policies, 34 interfaces, 5 VPN tunnels documented
- `university-network-architecture` — design-first documentation repository
- `network-automation-aruba` — automated Aruba campus switching

**What I learned:**
Documentation is an architectural artifact. The `university-network-architecture` repo is not a README — it is the authoritative record of why the network is the way it is. Without it, the next engineer inherits configuration without context, and configuration without context is technical debt.

---

## Chapter 3: Building the Platform (late 2025–early 2026)

Sinai University needed more than a network. It needed a platform — a coherent set of capabilities that development teams, operations teams, and the organization as a whole could rely on.

I designed and began implementing the **Nexus Platform** for Sinai University, mapping 14 core capability contracts to specific vendor implementations:

| Challenge | Decision | Why |
|-----------|----------|-----|
| Kubernetes distribution | RKE2 | FIPS compliance + rancher ecosystem integration |
| Network policy | Cilium + eBPF | Layer 7 visibility without service mesh complexity |
| GitOps engine | ArgoCD | Declarative sync with clear reconciliation model |
| Identity | Active Directory → Keycloak | Existing AD investment + modern OIDC broker |
| Firewall | FortiGate 1101E | Enterprise UTM with FortiManager integration |
| Cluster lifecycle | Cluster API (CAPV) | vSphere-native cluster management |

But the more important work was defining **what the platform is for**:

> *"Nexus is an organizational coordination system implemented with GitOps."*

This reframe changed everything. A platform is not a Kubernetes cluster. It is not a set of tools. It is the organizational surface through which teams coordinate around shared infrastructure — with explicit authority, observable outcomes, and traceable decisions.

**Key repos:**
- `nexus-platform-engineering` — 183 ADRs, vendor-agnostic governance core
- `nexus-platform-product-sinai-university` — Concrete implementation with 14 implementation ADRs
- `su_gitops_project` — GitOps manifests
- `university-network-architecture` — Network layer

---

## Chapter 4: Governing the Platform (early 2026–present)

A platform without governance degrades. Decisions get made for convenience. Authority becomes ambiguous. The platform drifts from its stated principles.

I designed three governance artifacts that prevent this:

### The Platform Operating System
A decision governance framework that externalizes and enforces decision quality. Key principle: *"Enforcement before convenience."* The system rejects shortcuts that bypass validation, manual overrides that replace automated evaluation, and implicit approvals through silence.

This is Principal-level thinking: not solving one decision problem, but designing a system that solves families of decision problems across time.

### The DC Migration Platform
A governance-first approach to datacenter migration. The problem with DC migrations isn't technical — it's organizational. Things fail because decisions are made verbally, approvals are assumed, and evidence is not collected. I built a repository structure that makes governance the default, not the overhead.

Key insight: *"Nothing is agreed unless it is documented. Nothing is executed unless it is approved."*

### Salwan Task OS
A personal operating system for managing complex, multi-domain technical work. Reflects the same principle applied at personal scale: files are truth, git is memory, diff before write.

---

## The Principal Difference

| Senior Platform Engineer | Principal Platform Engineer |
|--------------------------|------------------------------|
| Makes the platform work | Makes the platform trustworthy |
| Solves a problem | Designs a framework that solves a class of problems |
| Implements a decision | Documents the decision so it can be challenged |
| Owns the technology | Owns the governance model |
| Responds to incidents | Designs systems that prevent incident classes |
| Understands one site | Designs for multi-site independence |
| Builds the ADR | Builds the ADR *system* |

---

## What I'm Building Next

The Nexus Platform Core is in stewardship mode — architecturally complete, 183 ADRs, frozen for evolution without exceptional justification. The next work is:

1. **Phase B activation** — Moving from Phase A (Infrastructure Control) to Phase B (Explainable Platform) at Sinai University
2. **DC Migration execution** — Moving from governance design to active migration execution
3. **Platform Operating System Phase 10 completion** — Signal baseline establishment across all domains

---

*This is not a resume. It is an engineering story. The repositories are the evidence.*
