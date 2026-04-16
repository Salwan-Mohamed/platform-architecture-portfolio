# ADR-014: vSphere + CAPV as Cluster Lifecycle Platform

**Status:** Accepted  
**Date:** 2026-04-17  
**Source:** IMPL-0004 (vSphere as Virtualization Platform), IMPL-0014 (CAPV for Cluster Lifecycle)  
**Project:** Sinai University Platform

---

## Context

Sinai University operates on-premises with no cloud spend. The existing infrastructure is VMware vSphere. The platform requires Kubernetes clusters to be created, upgraded, and deleted as infrastructure operations — not as manual processes involving vSphere UI clicks and SSH sessions.

The naive approach: provision VMs manually, SSH in, run kubeadm or the RKE2 install script, join nodes by hand. This works for one cluster. It does not work when multiple clusters exist, when a cluster needs to be upgraded, or when a cluster needs to be rebuilt after a failure — all cases where manual operations produce inconsistent state and are not auditable.

IMPL-0014 defines Cluster API for vSphere (CAPV) as the answer: clusters defined as Kubernetes CRDs, lifecycle managed by a controller, all state in Git.

## Constraints

- No cloud infrastructure; vSphere is the only compute platform
- Budget prohibits replacing vSphere with an alternative hypervisor
- RKE2 is the chosen Kubernetes distribution (ADR-003) — CAPV must support RKE2
- Cluster lifecycle operations must be auditable via Git commit history
- Small ops team — manual cluster operations at scale are not sustainable

## Decision

**Use VMware vSphere as the compute platform. Use Cluster API for vSphere (CAPV) to manage RKE2 cluster lifecycle declaratively from Git.**

vSphere provides the VM infrastructure (compute, storage via vSAN, networking via distributed switches). CAPV runs on the management cluster as a controller. Cluster definitions — control plane nodes, worker nodes, vSphere resources, network config — are declared as Kubernetes CRDs committed to Git. CAPV reconciles from that desired state to actual vSphere infrastructure.

Cluster creation: commit a cluster CRD to Git → ArgoCD applies it → CAPV provisions VMs in vSphere → RKE2 bootstraps → cluster joins the fleet.  
Cluster upgrade: update the Kubernetes version field in Git → CAPV rolls the upgrade.  
Cluster deletion: delete the CRD from Git → CAPV deprovisions VMs.

No vSphere UI clicks are required for any cluster lifecycle operation after CAPV is operational.

## Alternatives Considered

**Manual cluster provisioning** — provision VMs via vSphere UI, bootstrap RKE2 manually. Rejected: not auditable, not repeatable at scale, inconsistent state across clusters, requires SSH access to nodes as an operational habit.

**Rancher for cluster management** — use the Rancher UI to provision and manage RKE2 clusters. Rejected: introduces a significant additional dependency (Rancher server), adds operational complexity, and moves cluster lifecycle out of Git into a UI. GitOps authority model requires cluster state to live in Git.

**OpenStack instead of vSphere** — replace VMware with open-source IaaS. Rejected: university has significant existing vSphere investment, licensing, and operational expertise. Migration cost is not justified by the benefit.

**Proxmox instead of vSphere** — lower-cost hypervisor. Rejected: no mature Cluster API provider; would require manual cluster operations or custom tooling.

## Trade-offs

**Accepted:** VMware licensing cost. vSphere is not free. The university carries existing licensing. This is an accepted constraint, not a choice made in this ADR.

**Accepted:** CAPV operational learning curve. CAPV adds a layer of abstraction over vSphere operations. Debugging a failed cluster provision requires understanding both Kubernetes CRD status conditions and vSphere events. Runbooks are required.

**Accepted:** CAPV is a dependency of cluster creation. If CAPV is unhealthy, no new clusters can be provisioned. Existing clusters continue operating — CAPV is not in the data path of running workloads, only in the provisioning path.

**Gained:** Cluster lifecycle is now a Git operation. Every cluster creation, upgrade, and deletion is a commit. Clusters can be reproduced exactly from their CRD definitions. Disaster recovery for a cluster is: restore the CRD to Git, CAPV recreates it.

## Consequences

- vSphere credentials are managed as Sealed Secrets in the management cluster
- All cluster definitions live in the GitOps repository under `gitops/clusters/`
- CAPV is deployed on the management cluster alongside ArgoCD
- A cluster template pattern is established for the first cluster and reused for all subsequent clusters
- Manual vSphere VM creation for Kubernetes nodes is prohibited after CAPV is operational
- Cluster upgrade procedures are defined as Git operations (update version field, observe CAPV rolling upgrade)

## Evidence

- IMPL-0004 accepted (Phase A)
- IMPL-0014 accepted (Phase A)
- RKE2 chosen in ADR-003 — CAPV's RKE2 bootstrap provider used
- Management cluster confirmed operational as CAPV host

## Principal-Level Signal

Cluster provisioning is infrastructure as code or it is technical debt. The first time you provision a cluster manually, you create an undocumented procedure. The tenth time, you create an operational dependency on the person who memorized the procedure. CAPV converts that procedure into a CRD. The cluster becomes a file. That is not a convenience — it is the difference between infrastructure you can audit and infrastructure you can only hope is correct.
