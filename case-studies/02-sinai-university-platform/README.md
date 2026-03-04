# Case Study 02 — Sinai University Platform Product

> *Implementing a production IDP for a multi-branch university using Nexus Platform Core*

**Repository**: [nexus-platform-product-sinai-university](https://github.com/Salwan-Mohamed/nexus-platform-product-sinai-university) (private)  
**Status**: Phase A Active · Phase B Partial · Phase C/D Planned  
**Core Compatibility**: v13.0 (127 Core ADRs referenced)

---

## Problem

Sinai University had infrastructure — servers, switches, VMs, firewalls — but no platform. No common way to provision environments, no shared identity layer, no standardized observability, no governance for how the infrastructure should evolve.

The result was the common pattern: each system was an island. Changes required manual coordination. Knowledge was held by individuals, not documented in systems. The infrastructure worked but couldn't be *reasoned about*.

---

## Environment

- **Organization**: University with 3 geographically distributed branches (Kantra, Katamia, Arish)
- **Existing infrastructure**: VMware vSphere compute, FortiGate firewalls, Aruba switching, Active Directory
- **Team**: Platform engineer (myself) + operations staff without platform engineering background
- **Constraint**: University procurement cycles — technology choices must work within existing vendor relationships
- **Scale**: Production workloads, academic calendar constraints (can't disrupt during exam periods)

---

## Constraints

- **Vendor compatibility**: Must integrate with existing FortiGate, VMware, and Active Directory — no greenfield
- **Skill transfer**: Platform must be operable by staff who are not Kubernetes-native
- **Progressive rollout**: Can't go from "no platform" to "full IDP" in one step — must phase delivery
- **Governance requirements**: University has audit and compliance requirements — all changes need traceability
- **Budget**: Public university constraints — open-source-first where possible

---

## Architecture Decision

### 14 Capabilities Mapped to Core Contracts

Every capability selection referenced a Core ADR. No tool was chosen because it was popular — each choice was justified against a capability contract.

| Core Capability | Implementation Choice | Decision Driver |
|-----------------|----------------------|----------------|
| Firewall Control Plane | FortiGate 1101E | Existing investment + FortiOS maturity |
| GitOps Reconciler | ArgoCD | Declarative model, strong RBAC, UI for ops team |
| Kubernetes Control Plane | RKE2 | FIPS compliance path, Air-gap capability |
| Virtualization Control Plane | VMware vSphere | Existing infrastructure, CAPV integration |
| Identity Authority | Active Directory | University's existing directory |
| Identity Broker | Keycloak | OIDC federation to AD, open-source, mature |
| Metrics Collection | Prometheus | Standard, operator ecosystem, Grafana native |
| Visualization | Grafana | Best-in-class dashboarding, AD/Keycloak integration |
| Log Aggregation | Loki | Grafana ecosystem, cost-effective vs Elasticsearch |
| Network Policy | Cilium | eBPF performance, L7 visibility, NetworkPolicy native |
| Infrastructure Provisioner | Terraform | State management, VMware provider maturity |
| Runbook Execution | AWX | Existing Ansible familiarity in ops team |
| Pipeline Execution | Tekton | Kubernetes-native, no external CI dependency |
| Cluster Lifecycle | CAPV (Cluster API) | VMware-native, declarative cluster management |

### Phase Delivery Model

```
Phase A (Active)   — Infrastructure Control
  ├── FortiGate deployed + managed via GitOps
  ├── VMware baseline automated via Terraform
  ├── RKE2 cluster operational
  ├── AWX runbooks for operational procedures
  └── Basic Prometheus/Grafana observability

Phase B (Partial)  — Explainable Platform
  ├── Keycloak identity broker live
  ├── Loki log aggregation deployed
  └── [ ] ArgoCD full application catalog pending

Phase C (Planned)  — Organizational Memory
  └── [ ] Service catalog + dependency mapping

Phase D (Future)   — Governed Self-Service
  └── [ ] Developer self-service golden paths
```

---

## Trade-offs

| Decision | Trade-off Made | Rationale |
|----------|----------------|----------|
| RKE2 over vanilla K8s | Rancher ecosystem coupling | Air-gap + FIPS capability outweighs lock-in |
| Keycloak over direct AD | Added identity layer | OIDC federation needed for modern apps |
| Loki over Elasticsearch | Less query flexibility | Cost and operational complexity |
| CAPV over manual VM provisioning | VMware API dependency | Declarative cluster lifecycle worth the coupling |
| AWX over pure GitOps for runbooks | Dual control planes | Ops team needs UI-driven runbook execution during transition |

---

## Outcome

- **Phase A fully operational** — infrastructure under GitOps control
- **14 capabilities mapped** to Core contracts with individual ADRs
- **Compliance checker script** validates product alignment with Core on every ADR change
- **Core/Product separation proven** — product ADRs reference Core, Core unmodified
- **Operations staff can execute runbooks** via AWX without platform engineering involvement

### What I Would Do Differently

1. **Deploy observability before applications** — Prometheus/Grafana should be Day 1, not retrofitted
2. **Keycloak federation in Phase A** — identity is a foundational capability, not Phase B
3. **Document every FortiGate policy** before automation — automation of undocumented state creates auditable chaos

---

## Operational Lessons

1. **Core/Product separation saves you when vendors change** — when FortiGate versioning changed behavior, only the product ADR needed updating, not any Core principles
2. **AWX is a transition tool, not a destination** — ops teams need it now, but the goal is GitOps-native runbooks
3. **CAPV gives cluster lifecycle management for free** — declaring a cluster in YAML and having it appear is worth the VMware coupling
4. **University procurement loves documented decisions** — having 14 ADRs with vendor rationale made budget approvals smoother
