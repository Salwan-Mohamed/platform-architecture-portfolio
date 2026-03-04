# ADR-003: RKE2 over Vanilla Kubernetes for University Platform

**Status:** Accepted  
**Date:** 2025-12-15  
**Decider:** Salwan Mohamed  
**Project:** Sinai University Platform — Kubernetes Layer  

---

## Context

Sinai University needed a production Kubernetes platform running on VMware vSphere (on-premises). The platform would host internal developer tools, monitoring stack (Prometheus/Grafana/Loki), identity services (Keycloak), and GitOps tooling (ArgoCD, Tekton). The ops team had limited Kubernetes operational experience and the university had compliance requirements around software supply chain and security hardening.

The decision was which Kubernetes distribution to run — and this choice would be foundational: everything else in the platform stack would be layered on top of it.

---

## Constraints

| Constraint | Impact |
|------------|--------|
| On-premises VMware vSphere only (no cloud) | Distribution must work well with CAPV (Cluster API Provider vSphere) |
| University compliance requirements | CIS benchmark compliance and FIPS-mode capability preferred |
| Small ops team (2-3 people managing clusters) | Minimal operational overhead for cluster lifecycle management |
| Air-gapped installation capability needed | Some university segments have restricted internet access |
| Must support Cilium as CNI | Network policy enforcement is a non-negotiable security requirement |
| Budget: no paid Kubernetes support contracts | Must be stable on open-source support tier |

---

## Decision

**Deploy RKE2 (Rancher Kubernetes Engine 2) as the Kubernetes distribution, provisioned via Cluster API with the vSphere provider (CAPV).**

RKE2 is chosen over vanilla Kubernetes (kubeadm), k3s, OpenShift, or Tanzu Kubernetes Grid.

---

## Alternatives Considered

| Option | Reason Rejected |
|--------|----------------|
| **Vanilla Kubernetes (kubeadm)** | No built-in security defaults; requires manual CIS hardening; no bundled components; higher operational burden for lifecycle management |
| **k3s** | Designed for edge/IoT; single-binary trade-offs create constraints at university scale; less suited for HA control plane on vSphere |
| **OpenShift** | Significant licensing cost; opinionated operator model conflicts with our GitOps-first approach; heavy resource overhead |
| **Tanzu Kubernetes Grid (TKG)** | VMware-native and vSphere-integrated, but requires VMware subscription; creates vendor lock-in at the Kubernetes layer conflicting with Core ADR-0139 (vendor-neutral architecture) |
| **EKS Anywhere** | AWS-aligned tooling creates conceptual mismatch for an org committed to on-prem; additional operational surface area |

---

## Trade-offs

**What we gained:**
- CIS Kubernetes Benchmark hardening by default — significant security baseline without manual effort
- FIPS 140-2 mode support for regulated university departments
- Air-gapped installation support with private registry configuration
- Integrated `rke2-server` / `rke2-agent` lifecycle model — simpler than kubeadm join tokens
- Strong community support and Rancher ecosystem integration if needed later
- Works natively with CAPV for declarative cluster lifecycle

**What we gave up:**
- RKE2 is one more abstraction layer over vanilla Kubernetes — debugging requires understanding both
- Rancher dependency: some RKE2 behaviors are specific to the Rancher ecosystem
- Upgrade cadence tied to Rancher's release cycle, slightly behind upstream Kubernetes

---

## Consequences

- `kubernetes-ha-cluster` repository implements the RKE2 HA cluster with 3 control plane nodes + worker pool on vSphere
- `ansible-role-k8s-ha-cluster` provides the Ansible automation for RKE2 deployment and configuration
- Cluster API (CAPV) manages cluster lifecycle declaratively from Git
- CIS benchmark compliance is validated post-deployment via automated scanning
- Cilium CNI is deployed on top of RKE2 without conflict

---

## Evidence

- [kubernetes-ha-cluster](https://github.com/Salwan-Mohamed/kubernetes-ha-cluster) — RKE2 HA cluster implementation
- [ansible-role-k8s-ha-cluster](https://github.com/Salwan-Mohamed/ansible-role-k8s-ha-cluster) — Automated RKE2 deployment
- [Sinai University IMPL-0003](https://github.com/Salwan-Mohamed/nexus-platform-product-sinai-university/blob/main/docs/01-decisions/adr/impl-0003-rke2-kubernetes.md) — Implementation ADR with full vSphere-specific rationale
- Core ADR-0139: Vendor-Neutral Architecture

---

## Principal-Level Signal

> Distribution selection is a 5-year decision. Choose based on your operational constraints and security baseline requirements — not based on what's trending. The question is not "which is the best Kubernetes" but "which Kubernetes can your team operate reliably at your security and compliance level, with the lifecycle tooling you already have?"
