# ADR-013: ArgoCD Hub-Spoke GitOps Architecture

**Status:** Accepted  
**Date:** 2026-04-17  
**Source:** IMPL-0002 (ArgoCD as GitOps Reconciler), IMPL-0017 (Pipeline Separation)  
**Project:** Sinai University Platform

---

## Context

ADR-002 established GitOps as the control plane and prohibited direct infrastructure changes. The implementation question — which GitOps tool, how deployed, how secured, how scaled to multiple clusters — was deferred to the implementation layer. IMPL-0002 resolves that question.

The platform requires continuous reconciliation across a management cluster and workload clusters. A single ArgoCD instance managing multiple clusters needs an explicit topology decision: should each cluster run its own ArgoCD instance, or should a central instance manage all clusters from a hub?

Secondary question: what separates the CI pipeline (build, test, image) from the CD pipeline (deploy, reconcile)? IMPL-0017 establishes that these are distinct systems with distinct authorities.

## Constraints

- Management cluster and workload clusters are distinct — platform services and workloads do not share a cluster
- ArgoCD must authenticate via Keycloak SSO; no local ArgoCD users
- RBAC must reflect AD group structure without per-cluster configuration duplication
- CI pipelines (Tekton) build artifacts; ArgoCD deploys them — these must never be the same system
- CRDs and security policies cannot be auto-pruned — operational risk of silent deletion

## Decision

**Deploy ArgoCD on the management cluster in HA (3 replicas). Manage all workload clusters from this hub using ApplicationSets. Never run ArgoCD on workload clusters.**

ArgoCD runs in the `argocd` namespace on the management cluster. It authenticates to workload clusters via kubeconfig credentials stored as Kubernetes secrets. ApplicationSets generate Applications across clusters from a single template, enabling consistent multi-cluster deployment without per-cluster ArgoCD configuration.

**Sync policies are resource-type-specific — not uniform across all resources:**

| Resource Type | Sync | Prune | Self-Heal | Rationale |
|---|---|---|---|---|
| Platform services | Auto | Yes | Yes | Drift from desired = incident |
| Observability stack | Auto | Yes | Yes | Monitoring must be correct |
| Security policies | Manual | No | Yes | Accidental deletion is a security event |
| Workload namespaces | Auto | Yes | Yes | Tenant isolation must be reconciled |
| CRDs | Manual | No | No | CRD deletion cascades to all instances |

**Pipeline separation:** Tekton runs CI (build → test → push image to registry). ArgoCD runs CD (detect new image tag in Git → reconcile to cluster). The boundary is Git: Tekton writes to Git (image tag update); ArgoCD reads from Git. Tekton never touches the cluster directly for deployment. ArgoCD never runs builds or tests.

**RBAC model maps AD groups to ArgoCD roles:**

| AD Group | ArgoCD Role | Capability |
|---|---|---|
| platform-admins | admin | Full access, all clusters |
| platform-operators | operator | Sync + view all apps |
| app-developers | developer | View + sync own applications |
| readonly | readonly | View only, no sync |

## Alternatives Considered

**Per-cluster ArgoCD instances** — each cluster manages its own GitOps reconciliation. Rejected: configuration duplication across clusters, no single pane for cross-cluster state, RBAC and SSO config multiplied per cluster, operational overhead linear with cluster count.

**Flux instead of ArgoCD** — CNCF GitOps alternative. Rejected for Sinai context: ArgoCD's UI provides operational visibility for a small team; ApplicationSets cover the multi-cluster requirement without additional tooling; the team's existing familiarity is with ArgoCD.

**Jenkins for CD** — push-based deployment from CI. Rejected: push-based requires cluster credentials stored in Jenkins; no drift detection; no self-healing; violates the GitOps authority model (Git, not Jenkins, is the source of truth for desired state).

## Trade-offs

**Accepted:** Management cluster is a critical dependency. If ArgoCD is unavailable, workload cluster reconciliation stops — existing deployments continue running, but no new deployments or drift corrections occur. Mitigation: HA deployment (3 replicas) with persistent state in cluster.

**Accepted:** Multi-cluster trust requires credential management. ArgoCD needs kubeconfig credentials for each workload cluster. These are stored as Sealed Secrets in Git, managed through the credential lifecycle defined in IMPL-0018.

**Gained:** Single pane across all clusters. Drift, sync status, and deployment history for every cluster in every namespace is visible from one ArgoCD UI. For a 4-person team, this operational consolidation is not optional.

## Consequences

- All Kubernetes state for all clusters flows through one ArgoCD instance
- New cluster onboarding = add to ApplicationSet generator, no new ArgoCD install
- Tekton builds never touch production clusters; ArgoCD never runs builds
- CRD management is explicitly manual — any CRD change requires human-initiated sync
- Security policy changes (NetworkPolicies, PodSecurityAdmission) require manual sync approval
- AD group changes propagate to ArgoCD RBAC via Keycloak without ArgoCD reconfiguration

## Evidence

- IMPL-0002 accepted 2026-01-23
- IMPL-0017 accepted (Phase A pipeline separation)
- Core ADR-0079 (Git as single control plane) and ADR-0082 (GitOps as sole authority) are the governing principles
- ArgoCD HA deployment confirmed in management cluster design

## Principal-Level Signal

Hub-spoke GitOps topology is a governance decision before it is a scaling decision. The question "where does ArgoCD run?" is really "who has the authority to reconcile cluster state, and how is that authority scoped?" A hub answers: one authority, consistent policy, single audit surface. Per-cluster instances answer: distributed authority, duplicated policy, N audit surfaces. For a platform that exists to provide consistent governance across sites, hub-spoke is the only coherent answer.
