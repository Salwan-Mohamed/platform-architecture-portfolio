# ADR-007: Full-Mesh VPN over Hub-and-Spoke for Multi-Site University

**Status:** Accepted  
**Date:** 2026-01-15  
**Decider:** Salwan Mohamed  
**Project:** Sinai University — Network Architecture  

---

## Context

Sinai University has 3 sites: Main Campus (Sinai), Branch Site 1, and Branch Site 2. All three sites need secure inter-site connectivity for:
- Active Directory domain services replication
- vSphere vMotion and VM storage replication
- Shared research data access
- Student and faculty cross-campus connectivity
- Platform management plane (ArgoCD, FortiManager reach all clusters)

The initial proposal was a hub-and-spoke topology with Main Campus as the hub. This would mean Branch1↔Branch2 traffic would traverse Main Campus, creating a single point of failure and a bandwidth bottleneck at the hub.

---

## Constraints

| Constraint | Impact |
|------------|--------|
| vSphere vMotion requires low-latency, high-bandwidth inter-site links | Hub-and-spoke doubles latency for Branch1↔Branch2 vMotion |
| AD replication must continue if Main Campus has a partial outage | Hub-and-spoke breaks AD replication between branches if hub is down |
| FortiGate 1101E at each site supports up to 10Gbps IPsec | Hardware capacity not the constraint |
| IP addressing: 3 sites with separate /16 address blocks | Route summarization is clean regardless of topology |
| Future SD-WAN: dual ISP at branch sites | Full-mesh is better positioned for SD-WAN path selection |
| Only 3 sites (not 10+) | Full-mesh complexity (N*(N-1)/2 tunnels) is manageable at this scale |

---

## Decision

**Deploy a full-mesh IPsec VPN topology with a direct tunnel between every site pair. For 3 sites, this requires 3 tunnels: Main↔Branch1, Main↔Branch2, Branch1↔Branch2.**

Routing: BGP over IPsec tunnels at each site, redistributing local site prefixes. Dynamic routing ensures that if one tunnel fails, traffic can reroute through the remaining topology.

---

## Alternatives Considered

| Option | Reason Rejected |
|--------|----------------|
| **Hub-and-spoke (Main Campus as hub)** | Single point of failure for inter-branch traffic; doubles latency for Branch1↔Branch2; vMotion bandwidth degradation; AD replication breaks if hub is degraded |
| **MPLS from ISP** | Significant recurring cost; requires ISP dependency for internal university traffic; limited flexibility for SD-WAN |
| **Overlay SD-WAN only (no IPsec)** | Insufficient encryption guarantees for HIPAA-adjacent research data in transit |
| **Hub-and-spoke with backup direct tunnel** | Hybrid approach is operationally complex — two routing models to maintain |

---

## Trade-offs

**What we gained:**
- Branch1↔Branch2 direct connectivity: no Main Campus dependency for inter-branch traffic
- AD replication resilience: branches can replicate directly even during Main Campus degradation
- vMotion performance: direct site-to-site path at wire speed, not double-hop through hub
- Route convergence: BGP over IPsec provides dynamic failover if any tunnel fails
- Future SD-WAN readiness: full-mesh topology maps naturally to SDWAN path policies

**What we gave up:**
- More tunnels to manage (3 vs 2 in hub-and-spoke)
- BGP configuration complexity — requires routing knowledge that is higher than static routes
- If a new site is added, complexity grows as N*(N-1)/2 — at 5+ sites, re-evaluate

---

## Consequences

- 3 FortiGate IPsec tunnels configured and operational between all sites
- BGP AS numbers assigned per site; eBGP peering over IPsec tunnels
- Route failover tested: Main↔Branch1 tunnel failure → Branch1 traffic reroutes via Branch1↔Branch2↔Main
- Monitoring: FortiAnalyzer alerts on tunnel state changes; Grafana dashboard shows BGP prefix counts per tunnel
- `university-network-architecture` documents complete topology, IP addressing, BGP configuration

---

## Evidence

- [university-network-architecture](https://github.com/Salwan-Mohamed/university-network-architecture) — Full network topology documentation
- [network-automation-aruba](https://github.com/Salwan-Mohamed/network-automation-aruba) — Campus switching layer automation
- Core ADR-0141: Capability-Driven Network Model
- Core ADR-0172: DNS Domain Strategy for Platform Services

---

## Principal-Level Signal

> Hub-and-spoke is the default topology because it's simple to draw and explain. Full-mesh is the right topology when the branches have meaningful direct communication requirements. The decision criterion is: **"Would a branch outage be caused or worsened by Main Campus being the only path?"** If yes, full-mesh. Always validate the topology against your actual traffic flows, not your network diagram aesthetic.
