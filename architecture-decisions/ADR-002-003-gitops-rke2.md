# ADR-002 — Git as Single Control Plane

**Status**: Accepted | **Date**: 2025-12 | **Author**: Salwan Mohamed

## Context
Infrastructure state drift is inevitable when multiple mechanisms can change state. Someone runs a manual command, a configuration changes in a UI, a vendor applies a patch — and declared state and actual state diverge. The question is not "should we automate" but "what is the single authority for what the infrastructure should be?"

## Decision
**Git is the single source of truth for all platform state. All changes flow through Git. State not in Git is not authoritative.**

```
Authority Chain:
  Git (declared state)
    └── ArgoCD (GitOps reconciler — continuously reconciles)
          └── Kubernetes clusters (actual state)

Rules:
  - Manual changes to cluster state are drift
  - Drift is detected and reported
  - Human approval for drift exceptions is documented in Git
```

## Consequences
**Positive**: Complete audit trail · Rollback = git revert · Drift detection built-in · PR workflow provides peer review for infrastructure changes  
**Negative**: Bootstrapping complexity · Secrets management requires additional tooling · Urgent changes still require Git workflow

## Alternatives Considered
| Alternative | Rejected Because |
|-------------|-----------------|
| Direct kubectl access | No audit trail; drift inevitable |
| UI-driven changes | No programmable interface; change history inaccessible |
| Ansible push-only | Declared state lives in playbooks, not queryable Git state |

## Signals
- ArgoCD sync status green across all applications
- Zero untracked manual changes detected in weekly drift reports
- All incidents traceable to specific Git commits

## References
- Core ADR-0079: Git Single Control Plane
- Product: ArgoCD as reconciler (IMPL-0002), Tekton as pipeline engine (IMPL-0013)


# ADR-003 — RKE2 as Kubernetes Distribution

**Status**: Accepted | **Date**: 2026-01 | **Author**: Salwan Mohamed

## Context
Choosing a Kubernetes distribution is not purely technical. The organization has limited platform engineering headcount, operations staff without Kubernetes-native background, air-gap requirements, potential FIPS compliance requirements, and existing VMware vSphere infrastructure requiring CAPV integration.

## Decision
**Use RKE2 (Rancher Kubernetes Engine 2) as the Kubernetes distribution.**

Key properties: FIPS 140-2 compliance path · Air-gap support built-in · CAPV (Cluster API Provider VMware) native integration · CIS Kubernetes Benchmark hardened by default · Embedded etcd for HA (no external etcd cluster required)

## Consequences
**Positive**: FIPS path without distribution change · Air-gap deployment without additional tooling · CAPV works natively · CIS hardening reduces audit burden  
**Negative**: Rancher/SUSE ecosystem coupling · Slower upstream version tracking vs k3s · RKE2-specific troubleshooting knowledge needed

## Alternatives Considered
| Alternative | Rejected Because |
|-------------|-----------------|
| kubeadm | High operational overhead; no air-gap built-in; no CAPV integration |
| k3s | Not FIPS-capable; less suitable for production multi-cluster |
| OpenShift | Too expensive; operator overhead too high for team size |
| EKS/GKE/AKS | On-premises requirement; cloud dependency unacceptable |

## Signals
- Air-gap deployment successfully tested in isolated network segment
- Cluster provisioned via CAPV with zero manual steps
- CIS benchmark score maintained across upgrades

## References
- Core ADR: Kubernetes Control Plane Capability Contract
- Product ADR: IMPL-0003
