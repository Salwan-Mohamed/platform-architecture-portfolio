# Case Study 01: Building a Platform Architecture for Sinai University

> **Role:** Platform Engineer (Solo Architect)  
> **Environment:** Sinai University — 3-site Egyptian university  
> **Duration:** 2025 Q4 → 2026 Q2  
> **Complexity:** ⭐⭐⭐⭐⭐  

---

## The Problem

Sinai University's IT infrastructure was operating in a state of controlled chaos. Three campus sites shared Active Directory but had no unified management plane. Changes were made via SSH and direct console access with no audit trail. A single network change in late 2024 caused a 40-minute multi-site outage — and after it was over, no one could definitively say who made the change, why, or what the previous state had been.

The university had acquired VMware vSphere licenses, FortiGate firewalls, and Aruba switching hardware — but these tools were each operated in isolation, without a unifying platform model. The question asked to me was: **"Can you build us something where we actually know what we have and control it?"**

That question became the Nexus Platform.

---

## The Environment

| Dimension | Detail |
|-----------|--------|
| **Sites** | 3 campuses: Main Campus (Sinai), Branch 1, Branch 2 |
| **Users** | ~5,000 students + 800 staff |
| **Infrastructure** | VMware vSphere, FortiGate 1101E, Aruba switching |
| **Kubernetes** | 0 clusters at start; target: 3-site RKE2 HA cluster |
| **Automation maturity** | Near-zero: manual CLI, no IaC, no GitOps |
| **Team** | 1 principal platform engineer + 3 site ops generalists |
| **Budget model** | University procurement cycles (annual, rigid) |
| **Compliance** | University governance + HIPAA-adjacent research data requirements |

---

## The Constraints

**Technical:**
- Existing hardware must be leveraged — no greenfield hardware procurement possible in year 1
- Air-gapped segments exist in the research network — online-only tooling is excluded
- vSphere is the hypervisor — cloud-native assumptions don't apply

**Organizational:**
- The ops team had strong networking skills but minimal Kubernetes or GitOps experience
- University IT governance required all changes to be auditable and approved
- Leadership would measure success by "can they explain what is running" — not by feature count

**Strategic:**
- The university had previously failed a "digital transformation" project that tried to do everything at once
- Any platform initiative needed to show value within 6 months or risk being cancelled

---

## The Architecture

### Layer 1: Governance Foundation
Before writing a single line of configuration, I established the architectural framework:

- **Nexus Platform Core** (`nexus-platform-engineering`): vendor-agnostic governance, 183 ADRs, defines *what* the platform must do without prescribing *how*
- **Sinai University Product** (`nexus-platform-product-sinai-university`): implements the Core with specific vendor choices (FortiGate, RKE2, ArgoCD, Keycloak, etc.)
- **Platform Operating System** (`platform-operating-system`): signal-driven governance framework ensuring decisions are evidence-based, not opinion-based

### Layer 2: Network Security Boundary
- FortiGate 1101E at each site as the primary security boundary
- Full-mesh IPsec VPN with BGP routing between all 3 sites
- FortiManager for centralized policy management
- Zero-trust segmentation between student, staff, research, and platform networks

### Layer 3: Virtualization & Compute
- VMware vSphere on existing hardware
- Cluster API (CAPV) for declarative VM lifecycle management
- Resource quotas and governance at the vSphere level before anything reaches Kubernetes

### Layer 4: Kubernetes Platform
- RKE2 for CIS-benchmarked, FIPS-capable Kubernetes
- Cilium as CNI for network policy enforcement and eBPF-based observability
- ArgoCD as GitOps reconciler — all cluster state lives in Git
- Tekton for CI pipeline execution

### Layer 5: Identity & Access
- Active Directory as the identity authority (existing investment preserved)
- Keycloak as the identity broker — OAuth2/OIDC bridge between AD and platform services
- Zero-trust: every platform service authenticates through Keycloak

### Layer 6: Observability (8 Axes)
- Prometheus (metrics), Grafana (visualization), Loki (logs)
- 8-axis observability framework: Infrastructure, Application, Service Health, Incident, Portfolio, Platform, FinOps, Business Operations

```
┌─────────────────────────────────────────────────────────┐
│           SINAI UNIVERSITY PLATFORM STACK               │
├─────────────────────────────────────────────────────────┤
│  Platform OS     │ Signal-driven governance              │
│  Nexus Core      │ 183 ADRs, vendor-agnostic doctrine    │
├─────────────────────────────────────────────────────────┤
│  ArgoCD          │ GitOps reconciliation                 │
│  Tekton          │ CI pipeline execution                 │
│  AWX             │ Runbook automation                    │
├─────────────────────────────────────────────────────────┤
│  RKE2            │ CIS-hardened Kubernetes               │
│  Cilium          │ Network policy + eBPF observability   │
│  CAPV            │ Declarative cluster lifecycle         │
├─────────────────────────────────────────────────────────┤
│  VMware vSphere  │ Virtualization layer                  │
│  FortiGate 1101E │ Security boundary + VPN               │
│  Aruba switching │ Campus network fabric                 │
├─────────────────────────────────────────────────────────┤
│  Keycloak        │ Identity broker (OAuth2/OIDC)         │
│  Active Directory│ Identity authority                    │
├─────────────────────────────────────────────────────────┤
│  Prometheus + Grafana + Loki │ 8-axis observability      │
└─────────────────────────────────────────────────────────┘
```

---

## The Key Decision Points

### Decision 1: Core/Product Separation
The most important architectural decision was made before any infrastructure was touched: separate the platform's governance principles (Core) from its environment-specific implementation (Product). This means Nexus Platform Core v14.0 contains 183 ADRs that are vendor-agnostic and portable. The Sinai University Product implements 14 of those contracts with specific vendor choices.

**Why it mattered:** When the university changes vendors or adds a fourth site, the governance layer doesn't need to be rebuilt. The Core stays; only the Product implementation changes.

### Decision 2: GitOps as the Change Control System
Rather than building a separate ITSM integration, I made Git the change control system. Every infrastructure change is a pull request with a named author, a commit message, and a review. ArgoCD enforces that the cluster state matches Git state.

**Why it mattered:** The university governance requirement for audit trails was satisfied by the Git commit history — no additional tooling needed.

### Decision 3: Phase A First, Features Later
I explicitly rejected the initial ask to build a developer self-service portal (Phase D capability) before the infrastructure was under version control (Phase A). This required a difficult conversation with leadership that paid off: the Phase A delivery (infrastructure under control, audit trail, monitored) was completed and demonstrably valuable before any Phase B/C/D features were discussed.

**Why it mattered:** The university had previously failed a project that tried to build everything at once. Phase-gating provided the psychological safety to say "this is enough for now" at each stage.

### Decision 4: Full-Mesh VPN over Hub-and-Spoke
The initial network design proposal used Main Campus as the hub. Analysis showed this would make Branch1↔Branch2 AD replication dependent on Main Campus availability — a single point of failure for inter-branch services. Full-mesh IPsec with BGP routing was more complex to configure but eliminated the topology-level SPOF.

**Why it mattered:** Two months after the VPN was deployed, Main Campus experienced a 4-hour internet outage. Branch sites continued to operate normally because their direct inter-site tunnel remained up.

---

## The Trade-offs

| Trade-off | What Was Given Up | What Was Gained |
|-----------|-----------------|----------------|
| Core/Product separation overhead | 2-3 weeks of upfront architecture design | Portable governance; no rebuild for new sites |
| GitOps discipline | "Just SSH in and fix it" speed | Full audit trail; university governance satisfied |
| Phase A before Phase D | Features delayed by ~6 months | Stable foundation; leadership confidence |
| RKE2 over vanilla k8s | Rancher ecosystem dependency | CIS hardening by default; FIPS support |
| FortiGate over Palo Alto | Best-in-class application inspection | 40% lower TCO; operational skills available in-region |

---

## The Outcome

- **Infrastructure under version control:** 100% of network and compute configuration managed via Git
- **Audit trail:** Every change traceable to a named committer with documented rationale
- **Zero unauthorized changes** to production systems since GitOps adoption
- **Multi-site resilience:** Branch site outage during Main Campus failure validated full-mesh design
- **Platform maturity:** Phase A complete; Phase B (explainable infrastructure) in progress
- **Governance artifact:** 183-ADR platform Core in STEWARDSHIP MODE — architecturally complete
- **Ops team uplift:** Site operators can make changes via PR review without requiring principal-level expertise

---

## Operational Lessons

**1. The governance framework is the product, not the tooling.**  
ArgoCD, FortiGate, and RKE2 are implementations. The platform is the set of principles, decision records, and governance contracts that make those tools operate as a coherent system. When the tooling changes, the governance doesn't.

**2. Phase-gating must be defended actively.**  
The pressure to add Phase D features before Phase A is solid is constant. The phase model only works if the architect is willing to say "not yet" and explain why the foundation matters more than the feature.

**3. Full-mesh VPN complexity is worth it at 3 sites.**  
The extra BGP configuration paid for itself the first time Main Campus had an outage. The break-even point for full-mesh complexity vs hub-and-spoke fragility is lower than most architects assume.

**4. Identity is always the hardest layer.**  
Keycloak + AD integration consumed more design time than the Kubernetes layer. Identity is the cross-cutting concern that touches every other platform component. Design it first, not last.

---

## What I Would Do Differently

1. **Start with the Signals Registry earlier.** I built the observability stack in Phase A but didn't formally define signal thresholds and fitness functions until Phase B. The Platform Operating System formalized this — it should have been day-one.

2. **Document the "boring" decisions.** I documented the major architectural decisions in ADRs but initially under-documented the operational micro-decisions ("why is this VLAN numbered this way?", "why is this ArgoCD sync window set to this interval?"). These create tribal knowledge debt.

3. **Invest in runbook automation earlier.** AWX runbook automation was Phase A planned but late Phase A delivered. Manual runbooks during the early months created exactly the undocumented-tribal-knowledge problem the platform was designed to prevent.

---

## Source Repositories

| Repository | Role |
|------------|------|
| [nexus-platform-engineering](https://github.com/Salwan-Mohamed/nexus-platform-engineering) | Platform Core — 183 ADRs |
| [nexus-platform-product-sinai-university](https://github.com/Salwan-Mohamed/nexus-platform-product-sinai-university) | Sinai University Product Implementation |
| [platform-operating-system](https://github.com/Salwan-Mohamed/platform-operating-system) | Signal-driven governance framework |
| [university-network-architecture](https://github.com/Salwan-Mohamed/university-network-architecture) | Network topology & security design |
| [su_gitops_project](https://github.com/Salwan-Mohamed/su_gitops_project) | GitOps implementation |
| [kubernetes-ha-cluster](https://github.com/Salwan-Mohamed/kubernetes-ha-cluster) | RKE2 HA cluster |
| [network-automation-aruba](https://github.com/Salwan-Mohamed/network-automation-aruba) | Campus switching automation |
