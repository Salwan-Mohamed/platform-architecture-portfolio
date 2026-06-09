# Portfolio Sync Guide

> How to keep the portfolio current as source repos evolve  
> Last updated: 2026-06-09

---

## Source Repo ADR Tracks

`nexus-platform-product-sinai-university` has three ADR tracks:

| Track | Path | Phase | Count | Description |
|-------|------|-------|-------|-------------|
| Phase A IMPLs | `docs/01-decisions/adr/impl-000X` | A | 20 | Tool decisions: FortiGate, ArgoCD, RKE2, vSphere, Keycloak, Prometheus, Grafana, Loki, Cilium, Terraform, AWX, Tekton, CAPV, Sealed Secrets, Robusta, pipelines, credentials, TDD, static analysis |
| Phase B IMPLs | `docs/adr/IMPL-00XX` | B | 11 | Phase B decisions: DNS, Storage, OIDC, RBAC, SLOs, Alerting, Phase gates, Execution params, CNI, Execution tooling, N-S exposure |
| ARCH series | `docs/adr/ARCH-000X` | Governance | 5 | Architectural governance models: Maturity model, IA positioning, Domain framework, Integration rule, Refactory-first |

---

## Complete ADR Map: Source → Portfolio

### Phase A Track (impl-0001 to impl-0020)

| Source IMPL | Topic | Portfolio ADR | Status |
|-------------|-------|---------------|--------|
| impl-0001 | FortiGate firewall | ADR-004 | ✅ |
| impl-0002 | ArgoCD GitOps | ADR-013 | ✅ |
| impl-0003 | RKE2 Kubernetes | ADR-003 | ✅ |
| impl-0004 | vSphere virtualization | ADR-014 | ✅ |
| impl-0005 | Active Directory identity | — | Covered by ADR-009 |
| impl-0006 | Keycloak identity broker | ADR-009 | ✅ |
| impl-0007 | Prometheus metrics | ADR-015 | ✅ |
| impl-0008 | Grafana visualization | ADR-015 | ✅ |
| impl-0009 | Loki log aggregation | ADR-015 | ✅ |
| impl-0010 | Cilium network policy | ADR-010 | ✅ |
| impl-0011 | Terraform provisioning | ADR-016 | ✅ |
| impl-0012 | AWX runbook automation | ADR-016 | ✅ |
| impl-0013 | Tekton CI pipelines | ADR-016 | ✅ |
| impl-0014 | CAPV cluster lifecycle | ADR-014 | ✅ |
| impl-0015 | Sealed Secrets | ADR-016 | ✅ |
| impl-0016 | Robusta alert enrichment | ADR-015 | ✅ |
| impl-0017 | Pipeline separation (CI/CD) | ADR-013 | ✅ |
| impl-0018 | Credential lifecycle | ADR-016 | ✅ |
| impl-0019 | Infrastructure TDD | ⏳ | ADR-022 candidate |
| impl-0020 | Static analysis | ⏳ | ADR-022 candidate |

### Phase B Track (IMPL-0021 to IMPL-0031)

| Source IMPL | Topic | Portfolio ADR | Status |
|-------------|-------|---------------|--------|
| IMPL-0021 | DNS domain strategy | ⏳ | ADR-023 candidate |
| IMPL-0022 | Storage class strategy (Longhorn) | ⏳ | ADR-023 candidate |
| IMPL-0023 | OIDC authentication | ADR-009 | ✅ |
| IMPL-0024 | Group-based authorization | ADR-009 | ✅ |
| IMPL-0025 | SLO operating model | ADR-011 | ✅ |
| IMPL-0026 | Alerting strategy | ADR-011 | ✅ |
| IMPL-0027 | Phase transition criteria | ADR-012 | ✅ |
| IMPL-0028 | Infrastructure execution parameters | — | Covered by ADR-016 |
| IMPL-0029 | CNI / Cilium | ADR-010 | ✅ |
| IMPL-0030 | Execution tooling strategy | ADR-016 | ✅ |
| IMPL-0031 | North-south exposure model | ⏳ | ADR-023 candidate |

### ARCH Track (ARCH-0001 to ARCH-0005)

| Source ARCH | Topic | Portfolio ADR | Status |
|-------------|-------|---------------|--------|
| ARCH-0001 | Capability Maturity Model | ADR-017 | ✅ Added 2026-06-09 |
| ARCH-0002 | Infrastructure Automation Positioning | ADR-018 | ✅ Added 2026-06-09 |
| ARCH-0003 | Domain Application Framework | ADR-019 | ✅ Added 2026-06-09 |
| ARCH-0004 | Integration Rule | ADR-020 | ✅ Added 2026-06-09 |
| ARCH-0005 | Refactory-First Platform Change Model | ADR-021 | ✅ Added 2026-06-09 |

---

## Remaining ADR Candidates

| Portfolio ADR | Source | Topic |
|---------------|--------|-------|
| ADR-022 | impl-0019, impl-0020 | Infrastructure TDD + Static Analysis |
| ADR-023 | IMPL-0021, IMPL-0022, IMPL-0031 | DNS Strategy + Longhorn Storage + North-South Exposure |

---

## Current Portfolio ADR Index

| ADR | Topic | Layer |
|-----|-------|-------|
| ADR-001 | Core/Product Separation | Governance |
| ADR-002 | GitOps as Control Plane | Governance |
| ADR-003 | RKE2 over Vanilla Kubernetes | Kubernetes |
| ADR-004 | FortiGate as Network Boundary | Network |
| ADR-005 | Phase-Based Platform Maturity | Governance |
| ADR-006 | Signal-Driven Decision Governance | Governance |
| ADR-007 | Full-Mesh VPN over Hub-Spoke | Network |
| ADR-008 | Human-in-Loop DC Migration | Governance |
| ADR-009 | Keycloak as Identity Broker | Identity |
| ADR-010 | Cilium CNI over Canal | Kubernetes |
| ADR-011 | SLO-Driven Alerting | Observability |
| ADR-012 | Evidence-Gated Phase Transitions | Governance |
| ADR-013 | ArgoCD Hub-Spoke GitOps | GitOps |
| ADR-014 | vSphere + CAPV Cluster Lifecycle | Infrastructure |
| ADR-015 | Four-Component Observability Stack | Observability |
| ADR-016 | Infrastructure Execution Stack | Infrastructure |
| ADR-017 | Capability Maturity Model | Governance |
| ADR-018 | Infrastructure Automation Positioning | Governance |
| ADR-019 | Domain Application Framework | Governance |
| ADR-020 | Integration Rule | Governance |
| ADR-021 | Refactory-First Platform Change Model | Governance |

---

## Next Sync: Commands for Claude Code

```bash
cd platform-architecture-portfolio
claude
```

```
# Check for new source IMPLs or ARCH ADRs
Discovery Agent: analyze nexus-platform-product-sinai-university

# Write remaining candidates
Architect Agent: write ADR-022 from impl-0019 infrastructure TDD and impl-0020 static analysis
Architect Agent: write ADR-023 from IMPL-0021 DNS strategy, IMPL-0022 Longhorn storage, IMPL-0031 north-south exposure

# Sync TaskOS diagrams still pending
Diagram Agent: read OBJ-20260416-diagram-identity-architecture-sinai from taskos vault and commit to system-diagrams/
Diagram Agent: read OBJ-20260416-diagram-cilium-hubble-observability-sinai from taskos vault and commit to system-diagrams/

# Update CS-01 with ARCH governance layer
Storyteller Agent: update CS-01 with ADR-017 through ADR-021 governance framework outcomes

# LinkedIn content from ARCH series
Positioning Agent: write LinkedIn post from ADR-017 capability maturity model
Positioning Agent: write LinkedIn post from ADR-018 infrastructure automation positioning
Positioning Agent: write LinkedIn post from ADR-021 refactory-first model
```
