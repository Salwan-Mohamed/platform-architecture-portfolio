# ADR-010: Cilium CNI Over Canal Default

**Status:** Accepted  
**Date:** 2026-04-16  
**Source:** IMPL-0029 (CNI / Network Stack Selection)  
**Project:** Sinai University Platform

---

## Context

RKE2 ships with Canal (Flannel + Calico) as the default Container Network Interface. When the platform was initially specified in Phase A, CNI selection was implicit — Canal was assumed because it was the RKE2 default. IMPL-0010 (Phase A) selected Cilium for network policy enforcement, but did not formally address the full CNI stack replacement.

During Phase B planning, the platform owner confirmed that Cilium is required as the full network substrate, not just the policy enforcement layer — and that Hubble is required alongside it for network-layer observability. This required a formal architectural amendment superseding the Canal default.

This is an architectural decision, not an execution detail. The choice of CNI determines the networking model (eBPF vs iptables), the policy language (CiliumNetworkPolicy vs Calico NetworkPolicy), the observability capability (Hubble vs none), and the long-term Gateway API readiness.

## Constraints

- RKE2 default (Canal) must be explicitly replaced — it does not co-exist with Cilium
- Ubuntu 22.04 is the node OS — eBPF kernel requirements are satisfied
- Team must learn Cilium operational tooling (cilium CLI, hubble CLI) — this is accepted cost
- OpenTelemetry / distributed tracing is explicitly deferred — Prometheus + Loki sufficient for Phase B
- Service mesh (Linkerd, Istio) is explicitly excluded — Cilium L7 network policy is sufficient

## Decision

**Replace Canal with Cilium as the CNI for the Sinai University Platform. Deploy Hubble alongside Cilium as a required network observability component.**

Cilium is deployed via Helm chart through Ansible automation immediately after RKE2 cluster bootstrap. Canal is not configured. Hubble UI and Hubble metrics (exported to Prometheus) are enabled as part of the same deployment.

CiliumNetworkPolicy replaces Calico NetworkPolicy as the policy language. All network policies going forward are written in CiliumNetworkPolicy format.

## Alternatives Considered

**Keep Canal (RKE2 default)** — accept the default and add Calico for policy. Rejected: no L7 policy capability, no native network observability, iptables-based performance ceiling, no Gateway API path. Operational simplicity does not outweigh these architectural limitations.

**Calico standalone CNI** — replace Flannel but keep Calico for both networking and policy. Rejected: still iptables-based, no Hubble-equivalent observability, weaker L7 capability than Cilium.

**Service mesh (Istio/Linkerd) for L7** — use Canal + service mesh for L7 policy. Rejected: adds significant operational complexity, sidecar overhead, and a second network policy model alongside Calico. Cilium provides L7 capability without service mesh overhead.

## Trade-offs

**Accepted:** Operational learning curve. The team must learn Cilium's tooling and debugging model. eBPF-based troubleshooting is different from iptables-based — `cilium monitor`, `hubble observe`, and the Hubble UI replace `iptables -L` and tcpdump. Mitigation: documentation and runbooks created before Phase C.

**Accepted:** Deployment complexity. Cilium requires explicit Helm values configuration and must be deployed after RKE2 but before workloads. Implementation Units in the Phase B plan are updated to reflect this dependency chain.

**Gained:** Network observability by default. Hubble provides per-flow network visibility (source, destination, verdict, policy reason) that is simply not available with Canal. Every dropped packet has an observable reason. This changes network troubleshooting from art to science.

**Gained:** L7-aware network policy. CiliumNetworkPolicy can restrict traffic at the HTTP method/path level, not just IP:port. This closes a security boundary that iptables-based CNIs cannot address.

## Consequences

- Canal is not deployed on this cluster — Cilium is the only CNI
- All network policies are written as CiliumNetworkPolicy resources
- Hubble metrics appear in Prometheus; Hubble dashboards are added to Grafana
- Observability stack gains a fourth signal: network flows (alongside metrics, logs, and traces-deferred)
- Future Gateway API adoption is enabled — Cilium is a certified Gateway API implementation
- OpenTelemetry and distributed tracing remain explicitly deferred; revisit at Phase C gate

## Evidence

- IMPL-0029 approved 2026-02-04 by Human Owner
- Ubuntu 22.04 confirmed as node OS — Linux kernel 5.15+, eBPF requirements satisfied
- IMPL-0010 (Phase A) had already selected Cilium for policy enforcement — this ADR formalises the full CNI replacement
- IMPL-0031 (North-South exposure) confirms Gateway API as the future path — Cilium Gateway API support is architectural enabler

## Principal-Level Signal

Default CNI is a technology convenience, not an architectural decision. Accepting a default is still a decision — it just means the constraints and trade-offs were never made explicit. When the CNI determines your observability model, your policy language, your performance ceiling, and your long-term feature roadmap, the right treatment is a formal ADR, not an implicit accept. The 30 minutes to write this decision pays dividends every time someone asks why the platform uses Cilium instead of what came in the box.
