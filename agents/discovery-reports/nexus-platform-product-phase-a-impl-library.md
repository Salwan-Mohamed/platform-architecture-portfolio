# Discovery Report: Sinai University Phase A IMPL Library (impl-0001 to impl-0020)

> **Agent:** Discovery Agent  
> **Date:** 2026-04-17  
> **Source:** nexus-platform-product-sinai-university/docs/01-decisions/adr/  
> **Status:** Previously undiscovered — full Phase A IMPL library not yet in portfolio

---

## What This Report Covers

The source repo has two distinct ADR tracks:

| Track | Path | Range | Phase | Portfolio Status |
|-------|------|-------|-------|------------------|
| Phase A IMPLs | `docs/01-decisions/adr/` | impl-0001–0020 | Phase A | ❌ Not synced |
| Phase B IMPLs | `docs/adr/` | IMPL-0021–0031 | Phase B | ✅ Partially synced (ADR-009–012) |

This report covers the Phase A library (impl-0001–0020) for the first time.

---

## Complete Phase A IMPL Inventory

| IMPL | Title | Size | Portfolio ADR Candidate |
|------|-------|------|-------------------------|
| impl-0001 | FortiGate as Firewall Platform | 9KB | → ADR-004 (already exists, needs enrichment) |
| impl-0002 | ArgoCD as GitOps Reconciler | 11KB | → ADR-013 (new — hub-spoke, RBAC, sync policies) |
| impl-0003 | RKE2 as Kubernetes Control Plane | 11KB | → ADR-003 (already exists, needs enrichment) |
| impl-0004 | vSphere as Virtualization Platform | 7KB | → ADR-014 (new) |
| impl-0005 | Active Directory as Identity Source | 8KB | → ADR-009 enrichment |
| impl-0006 | Keycloak as Identity Broker | 11KB | → ADR-009 (already exists, confirms decision) |
| impl-0007 | Prometheus as Metrics Backend | 9KB | → ADR-015 (new) |
| impl-0008 | Grafana as Visualization Layer | 3KB | → ADR-015 enrichment |
| impl-0009 | Loki as Log Aggregation Backend | 8KB | → ADR-015 enrichment |
| impl-0010 | Cilium for Network Policy Enforcement | 7KB | → ADR-010 (already exists, confirms decision) |
| impl-0011 | Terraform for Infrastructure Provisioning | 9KB | → ADR-016 (new) |
| impl-0012 | AWX for Runbook Automation | 11KB | → ADR-016 enrichment |
| impl-0013 | Tekton for CI Pipelines | 5KB | → ADR-016 enrichment |
| impl-0014 | CAPV for Cluster Lifecycle | 8KB | → ADR-014 enrichment |
| impl-0015 | Sealed Secrets for Secret Management | 8KB | → ADR-016 enrichment |
| impl-0016 | Robusta for Alert Enrichment | 7KB | → ADR-011 enrichment |
| impl-0017 | Pipeline Separation (CI vs CD) | 8KB | → ADR-013 enrichment |
| impl-0018 | Credential Lifecycle Management | 6KB | → ADR-016 enrichment |
| impl-0019 | Infrastructure TDD | 6KB | → new ADR candidate |
| impl-0020 | Static Analysis | 8KB | → new ADR candidate |

---

## Key Architectural Signals Not Yet in Portfolio

### ArgoCD Hub-Spoke Model (impl-0002)
- Management cluster hosts ArgoCD in HA (3 replicas)
- ApplicationSets manage multi-cluster deployment patterns
- Sync policies differ by resource type: platform services auto-sync + self-heal; CRDs manual only; security policies manual prune
- RBAC model: AD group `platform-admins` → ArgoCD admin; `platform-operators` → sync all; `app-developers` → sync own; `readonly` → view
- ArgoCD authenticated via Keycloak SSO (not local users)

### Observability Stack Composition (impl-0007, -0008, -0009, -0016)
- **Prometheus**: metrics backend, multi-cluster scraping via federation or remote write
- **Grafana**: visualization, AD group-based dashboard access
- **Loki**: log aggregation, Promtail agents on all nodes
- **Robusta**: alert enrichment layer — adds context (runbook links, pod logs, events) to Alertmanager alerts before routing
- Together these form a 4-component observability stack with Hubble as the fifth (network layer)

### Infrastructure Execution Stack (impl-0011, -0012, -0013)
- **Terraform**: provisions vSphere VMs, network objects, DNS records — infrastructure only
- **AWX**: executes Ansible playbooks for OS config, cluster bootstrapping, application config
- **Tekton**: CI pipelines for build, test, image push — runs in-cluster
- Clear separation: Terraform = infrastructure state; AWX = configuration state; Tekton = artifact state

### Cluster Lifecycle (impl-0014)
- **CAPV** (Cluster API for vSphere): manages RKE2 cluster lifecycle declaratively
- Clusters defined as CRDs in Git; CAPV reconciles to desired state
- Enables cluster creation, upgrade, and deletion without manual vSphere operations

### Secret Management (impl-0015)
- **Sealed Secrets**: asymmetric encryption; public key encrypts, controller private key decrypts
- SealedSecret CRDs committed to Git safely; decrypted Secrets never in Git
- Addresses the core GitOps secret problem: how to store encrypted secrets in the same repo as everything else

---

## Portfolio ADRs to Generate from This Report

| Portfolio ADR | Source IMPLs | Topic |
|---------------|--------------|-------|
| ADR-013 | impl-0002, impl-0017 | ArgoCD Hub-Spoke GitOps Architecture |
| ADR-014 | impl-0004, impl-0014 | vSphere + CAPV as Cluster Lifecycle Platform |
| ADR-015 | impl-0007, impl-0008, impl-0009, impl-0016 | Four-Component Observability Stack |
| ADR-016 | impl-0011, impl-0012, impl-0013, impl-0015, impl-0018 | Infrastructure Execution Stack |

---

## Claude Code Agent Output — Sync Status

The Claude Code agent report (2026-04-16) indicates 9 files were created in the TaskOS vault:
- `writing/WRT-20260416-discovery-nexus-platform-sinai-university.md`
- `decisions/DEC-20260416-adopt-cilium-cni.md`
- `decisions/DEC-20260416-spiffe-workload-identity.md`
- `decisions/DEC-20260416-entra-id-unified-idp.md`
- `decisions/DEC-20260416-gitops-argocd-flux.md`
- `decisions/DEC-20260416-adopt-hubble-observability.md`
- `writing/WRT-20260416-cs01-sinai-university-platform.md`
- `objects/OBJ-20260416-diagram-identity-architecture-sinai.md`
- `objects/OBJ-20260416-diagram-cilium-hubble-observability-sinai.md`

These are in the TaskOS vault format. To sync to portfolio: use `taskos_read_file` for each and push to the portfolio repo.

**Note on ADR content in Claude Code output:** The agent proposed SPIFFE/SPIRE and Entra ID as decisions. These are not yet in the source IMPL library — they appear to be forward-looking recommendations, not current decisions. Treat as ADR candidates pending human review, not as decisions to add to the portfolio ADR library directly.
