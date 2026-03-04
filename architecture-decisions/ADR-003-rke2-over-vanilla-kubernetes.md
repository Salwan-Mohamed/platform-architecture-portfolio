# ADR-003: RKE2 over Vanilla Kubernetes for University Production

**Status:** Accepted  
**Date:** 2025-12-01  
**Decider:** Salwan Mohamed  
**Domain:** Infrastructure  
**Project:** Nexus Platform — Sinai University Implementation

---

## Context

Sinai University requires a production Kubernetes platform that is maintainable by a small team, compliant with institutional security requirements, and integrated with the existing VMware vSphere infrastructure. The choice of Kubernetes distribution is a foundational infrastructure decision that affects every other layer of the platform.

Vanilla Kubernetes (kubeadm) requires significant manual work to produce a production-ready cluster: etcd management, certificate rotation, component upgrades, and security hardening are all manual processes.

---

## Constraints

- Small operations team (1-2 engineers) — operational overhead must be minimal
- VMware vSphere is the existing virtualization layer — integration is required
- Institutional security requirements — FIPS 140-2 compliance is a consideration
- No managed Kubernetes available (no AWS/Azure/GCP) — self-managed only
- Rancher is already part of the evaluation landscape for management UI

---

## Decision

**Use RKE2 (Rancher Kubernetes Engine 2) as the Kubernetes distribution for Sinai University production clusters, with Cluster API Provider vSphere (CAPV) for cluster lifecycle management.**

---

## Alternatives Considered

| Option | Reason Rejected |
|--------|-----------------|
| Vanilla Kubernetes (kubeadm) | Too much operational overhead for small team; no integrated security hardening; manual upgrade process |
| K3s | Designed for edge/lightweight scenarios; not suitable for full production workloads with enterprise features |
| OpenShift | Too expensive for university budget; Red Hat licensing overhead; over-engineered for this scale |
| Tanzu (VMware) | Expensive additional licensing on top of existing vSphere; limited community support; vendor lock-in risk |
| k0s | Newer project, less ecosystem maturity; limited support resources |

---

## Trade-offs

**Given up:**
- Pure upstream Kubernetes — RKE2 adds Rancher-specific components
- Flexibility of rolling your own distribution configuration

**Gained:**
- FIPS 140-2 compliance built-in (critical for institutional environments)
- CIS Kubernetes Benchmark compliance by default
- Simplified upgrade process (single binary, rolling upgrades)
- Native vSphere integration via CAPV
- Strong security defaults (no anonymous auth, audit logging enabled, network policies enforced)
- Rancher management UI available if desired

---

## Consequences

**Positive:**
- RKE2 clusters deployed via Cluster API (CAPV) — cluster lifecycle managed as code
- Security baseline is maintained automatically — operators can't accidentally disable security features
- Cilium CNI integrates cleanly with RKE2's CNI model
- Upgrade path is tested and documented by Rancher — reduces operational risk for small team

**Negative:**
- CAPV adds complexity to the cluster provisioning workflow
- RKE2-specific bugs require Rancher upstream tracking, not vanilla Kubernetes tracking
- Dependency on Rancher's release cadence for upstream Kubernetes version support

---

## Evidence

- [nexus-platform-product-sinai-university IMPL-0003](https://github.com/Salwan-Mohamed/nexus-platform-product-sinai-university/blob/main/docs/01-decisions/adr/impl-0003-rke2-kubernetes.md)
- [kubernetes-ha-cluster](https://github.com/Salwan-Mohamed/kubernetes-ha-cluster) — HA cluster implementation reference
- [kubernetes-production-journey](https://github.com/Salwan-Mohamed/kubernetes-production-journey) — Day 0/1/2 production guide developed during this implementation
- [cilium-deep-dive-labs](https://github.com/Salwan-Mohamed/cilium-deep-dive-labs) — CNI evaluation work

---

## Operational Lesson

The CAPV integration revealed an important dependency ordering constraint: vSphere credentials must be available to CAPV before cluster provisioning can begin. This creates a chicken-and-egg problem when bootstrapping a new environment from scratch. The solution was a bootstrap cluster (local kind cluster) that runs CAPV and manages the production cluster's lifecycle — a pattern that became an ADR in the Nexus Core.

---

*Reference Core ADR: nexus-platform-engineering/docs/01-decisions/adr/impl-0003-rke2-kubernetes.md*  
*Reference Core Capability: Kubernetes Control Plane*
