# Salwan Mohamed — Platform Architecture Portfolio

> **Senior → Principal Platform Engineer**  
> Building structured infrastructure platforms for multi-site organizations

[![GitHub](https://img.shields.io/badge/GitHub-Salwan--Mohamed-black)](https://github.com/Salwan-Mohamed)

---

## Engineering Identity

I build platforms that organizations can *reason about*. Not just infrastructure that runs — but systems where every decision is traceable, every capability is explicit, and every team member knows who owns what.

My work centers on one problem: **most organizations have infrastructure. Few have a platform.** The difference is governance, intentionality, and the discipline to document decisions before they evaporate.

### What I Build

| Domain | What I Do |
|--------|-----------|
| **Platform Architecture** | Vendor-agnostic governance foundations with ADR-driven design |
| **University Infrastructure** | Multi-site platforms across 3 geographically distributed branches |
| **Datacenter Migration** | Evidence-based, gate-controlled migration programs |
| **GitOps & IDP** | RKE2 + ArgoCD + Tekton + Cluster API production platforms |
| **Network Architecture** | FortiGate full-mesh IPsec, zero-trust segmentation, multi-site VPN |
| **Automation Engineering** | AWX/Ansible for network and infrastructure at scale |

---

## Portfolio Map

```
platform-architecture-portfolio/
│
├── README.md                          ← You are here
├── CAREER-NARRATIVE.md                ← Senior → Principal trajectory
│
├── case-studies/                      ← The work that defines the career
│   ├── 01-nexus-platform-core/        ← 183 ADRs, vendor-agnostic platform foundation
│   ├── 02-sinai-university-platform/  ← 14-capability production IDP
│   ├── 03-university-network-arch/    ← Full-mesh IPsec, 3-branch university
│   ├── 04-dc-migration-platform/      ← Evidence-based migration governance
│   └── 05-platform-operating-system/  ← Signal-driven decision meta-governance
│
├── architecture-decisions/            ← ADR library (Principal-level thinking)
│   ├── ADR-001  Core/Product Separation
│   ├── ADR-002  GitOps as Control Plane
│   ├── ADR-003  RKE2 Kubernetes Distribution
│   ├── ADR-004  FortiGate Network Control Plane
│   ├── ADR-005  Vendor-Agnostic Core Architecture
│   ├── ADR-006  Phase-as-Product Model
│   ├── ADR-007  Human-Gated Decision Authority
│   └── ADR-008  Full-Mesh VPN Topology
│
├── system-diagrams/                   ← Architecture as code (Mermaid)
│   ├── nexus-platform-layers.md
│   ├── university-multi-site-topology.md
│   └── platform-authority-model.md
│
├── governance/
│   ├── platform-governance-model.md
│   └── multi-site-operations-model.md
│
├── linkedin/
│   ├── headline-and-about.md
│   └── project-descriptions.md
│
└── .claude/agents/                    ← AI agent roles for portfolio maintenance
    ├── discovery-agent.md
    ├── adr-writer-agent.md
    ├── case-study-writer-agent.md
    ├── linkedin-writer-agent.md
    └── diagram-agent.md
```

---

## Key Projects

| Project | Scale | Technologies | Impact |
|---------|-------|-------------|--------|
| [Nexus Platform Core](case-studies/01-nexus-platform-core/README.md) | 183 ADRs, 4 phases | GitOps, Architecture Governance | Vendor-agnostic platform foundation for any organization |
| [Sinai University Platform](case-studies/02-sinai-university-platform/README.md) | 3 sites, 14 capabilities | RKE2, ArgoCD, FortiGate, VMware, Keycloak | Production IDP for multi-branch university |
| [University Network Architecture](case-studies/03-university-network-arch/README.md) | 3 branches, full-mesh VPN | FortiGate-1101E, IPsec, VLAN | Zero-hub-dependency branch connectivity |
| [DC Migration Platform](case-studies/04-dc-migration-platform/README.md) | Enterprise program | Governance, Evidence, Gate-control | Human-gated evidence-based migration framework |
| [Platform Operating System](case-studies/05-platform-operating-system/README.md) | Decision governance | Signal-driven, ADR-gated | Prevents decision quality degradation over time |

---

## Technology Fingerprint

**Compute**: VMware vSphere · RKE2 · Cluster API (CAPV)  
**Networking**: FortiGate · Cilium · IPsec · VLAN · Zero-Trust  
**GitOps**: ArgoCD · Tekton · Terraform · Ansible · AWX  
**Observability**: Prometheus · Grafana · Loki · 8-Axis Framework  
**Identity**: Active Directory · Keycloak · OIDC  
**Governance**: ADR methodology · Phase-as-Product · Authority Models  

---

*"Most organizations have infrastructure. I build platforms — systems you can reason about, govern, and evolve with confidence."*
