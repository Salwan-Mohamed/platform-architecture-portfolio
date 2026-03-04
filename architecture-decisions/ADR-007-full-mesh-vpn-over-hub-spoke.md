# ADR-007: Full-Mesh VPN over Hub-Spoke for University Branch Connectivity

**Status:** Accepted  
**Date:** 2025-12-29  
**Decider:** Salwan Mohamed  
**Domain:** Network  
**Project:** Sinai University Network Architecture

---

## Context

Sinai University has three branch campuses (Kantra, Katamia, Arish) and a central datacenter. Branches need to communicate with each other for shared services (file shares, voice, inter-campus applications). The default network engineering response to this topology is hub-spoke: all inter-branch traffic routes through the datacenter.

I rejected hub-spoke. This ADR documents why.

---

## Constraints

- 3 branch sites, each with local internet breakout
- Datacenter is NOT always available — planned maintenance, unplanned outages
- Branch-to-branch traffic (voice, file sharing) is latency-sensitive
- Each site has a FortiGate edge firewall capable of terminating multiple VPN tunnels
- Small network team — VPN topology must be operationally maintainable

---

## Decision

**Deploy direct site-to-site IPsec tunnels between ALL branch edge firewalls (full-mesh), with each branch maintaining local internet breakout. No branch traffic is forced through the datacenter for inter-branch connectivity.**

```
      Kantra-site ◄─────────► Katamia-site
          ▲                       ▲
          │                       │
          └───────► Arish-site ◄──┘
```

Result: 3 direct tunnels, 0 hub dependencies.

---

## Alternatives Considered

| Option | Reason Rejected |
|--------|-----------------|
| Hub-spoke through datacenter | Datacenter outage = all inter-branch connectivity fails; adds latency for local traffic; creates bottleneck |
| Hub-spoke through primary branch (Kantra) | Primary branch becomes single point of failure for entire network; takes Kantra down = network partition |
| SD-WAN overlay | Budget and operational complexity exceed requirements at current scale |
| MPLS from ISP | Cost and lead time excessive; full-mesh IPsec achieves same result with existing equipment |

---

## Trade-offs

**Given up:**
- Centralized traffic inspection for all inter-branch traffic (hub-spoke enables this)
- Simpler tunnel count at datacenter (hub-spoke concentrates tunnels centrally)

**Gained:**
- Branch independence: Kantra can reach Arish even if datacenter is down
- Lower latency for inter-branch traffic (direct path, not via datacenter)
- No single point of failure in the network topology
- Datacenter maintenance windows don't affect branch-to-branch connectivity
- Each branch has local internet breakout — SaaS access doesn't traverse the datacenter

---

## Consequences

**Positive:**
- Arish ↔ Kantra tunnel: ✅ Up and passing traffic
- SU_Remote (remote access VPN): ✅ Operational
- Branch internet access is autonomous — branches are not dependent on datacenter for outbound connectivity

**Negative:**
- Cairo-S2S (Kantra ↔ Cairo/Katamia): ⚠️ Phase 2 failure — 1,135 IPsec SA negotiations failed
  - This is a configuration/implementation issue, not an architectural flaw in the full-mesh design
  - Root cause under investigation
- N*(N-1)/2 tunnels scale quadratically — at 3 sites this is 3 tunnels; at 10 sites this is 45 tunnels (SD-WAN becomes appropriate at higher site counts)

---

## Scaling Note

Full-mesh VPN is appropriate for small-scale deployments (up to ~5-6 sites). At higher site counts, the operational overhead of maintaining N*(N-1)/2 tunnels becomes significant. The decision to use full-mesh is appropriate for Sinai University at current scale but should be re-evaluated if the university expands to additional campuses.

---

## Evidence

- [university-network-architecture](https://github.com/Salwan-Mohamed/university-network-architecture) — Full network architecture documentation
- Kantra-site VPN documentation: 5 tunnels documented, 3 active, 2 inactive
- Traffic statistics: 3.4GB RX / 729MB TX via VPN (as of December 28, 2025)

---

## Operational Lesson

The most important question in network topology design is: *"What happens to this network when X fails?"* Hub-spoke's answer to "What happens when the datacenter fails?" is "Inter-branch connectivity stops." Full-mesh's answer is "Inter-branch connectivity continues; only datacenter-hosted services are unreachable." For a university where students and faculty need to reach each other across campuses, the second answer is the right answer.

---

*This ADR documents a network topology decision with organizational resilience implications.*
