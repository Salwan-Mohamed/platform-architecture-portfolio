# ADR-004: FortiGate 1101E as the Network Security Boundary

**Status:** Accepted  
**Date:** 2026-01-05  
**Decider:** Salwan Mohamed  
**Project:** Sinai University Platform — Network Security Layer  

---

## Context

Sinai University has 3 campus sites (main campus + 2 branch sites), each requiring firewall enforcement, inter-site VPN connectivity, and perimeter security. The university was operating with aging layer-3 switches handling rudimentary ACLs as "firewalls" — no application-layer inspection, no centralized policy management, no IPS/IDS, and no visibility into east-west traffic between sites.

This created a significant security exposure: inter-site traffic was encrypted only by IPsec tunnels with pre-shared keys that had never been rotated, and there was no way to enforce policy on traffic between university departments without manually configuring ACLs on 30+ switches.

The platform required a centralized, manageable, policy-enforced security boundary at each site.

---

## Constraints

| Constraint | Impact |
|------------|--------|
| 3 physical sites, each needing a firewall | Must support centralized management across sites |
| University compliance: HIPAA-adjacent for medical research data | Application-layer inspection required; not just L3/L4 ACLs |
| Existing FortiGate familiarity in regional IT community | Operational skills available; reduces training overhead |
| Budget: mid-enterprise tier (not carrier-grade) | Rules out Palo Alto at full list price; Cisco ASA lacks modern featureset |
| Must support full-mesh IPsec VPN between 3 sites | All sites must be able to communicate directly (not hub-and-spoke) |
| SD-WAN readiness for future multi-ISP at branch sites | Future requirement; architecture must not preclude it |

---

## Decision

**Deploy FortiGate 1101E at each site as the primary network security boundary, managed via FortiManager for centralized policy, with full-mesh IPsec VPN between all three sites.**

FortiGate satisfies the Nexus Core "Firewall Control Plane" capability contract (IMPL-0001).

---

## Alternatives Considered

| Option | Reason Rejected |
|--------|----------------|
| **Palo Alto PA-Series** | Superior application inspection, but 40-60% higher TCO at this scale; no existing operational familiarity; harder to self-manage without PANW support contract |
| **Cisco ASA / Firepower** | Legacy ASA platform; Firepower migration complexity; Cisco's transition strategy creates long-term uncertainty; higher CLI complexity for the ops team |
| **pfSense/OPNsense** | Open-source community support insufficient for university compliance requirements; no centralized management; limited enterprise feature set |
| **Juniper SRX** | Strong technically, but no regional operational expertise; vendor support in region is limited |
| **Meraki MX** | Cloud-managed (Cisco Meraki); requires permanent internet connectivity for management plane; unacceptable for offline resilience; license model too expensive per-site |

---

## Trade-offs

**What we gained:**
- FortiManager provides single-pane-of-glass policy management across all 3 sites
- FortiGate 1101E provides 10Gbps firewall throughput — headroom for 5+ years of growth
- Built-in SD-WAN capability satisfies future multi-ISP requirement without hardware change
- SSL inspection, IPS, and application control satisfy compliance layer requirements
- FortiAnalyzer provides log aggregation and compliance reporting from all sites
- Full-mesh IPsec with BGP-over-VPN enables any-to-any site communication with route failover

**What we gave up:**
- FortiGate configuration-as-code is less mature than Palo Alto's Panorama/Terraform provider
- Vendor lock-in to Fortinet ecosystem (FortiManager, FortiAnalyzer, FortiSwitch integration)
- FortiGate CLI is idiosyncratic — learning curve for ops team members familiar with Cisco IOS

---

## Consequences

- 3x FortiGate 1101E deployed (1 per site) — active-passive HA pairs at main campus
- Full-mesh IPsec VPN established: 3 tunnels (Main↔Branch1, Main↔Branch2, Branch1↔Branch2)
- FortiManager deployed as centralized management platform
- `university-network-architecture` repository documents the complete FortiGate topology and policy design
- Network automation for FortiGate policy-as-code is in progress via Terraform FortiOS provider

---

## Evidence

- [university-network-architecture](https://github.com/Salwan-Mohamed/university-network-architecture) — Network security architecture documentation
- [Sinai University IMPL-0001](https://github.com/Salwan-Mohamed/nexus-platform-product-sinai-university/blob/main/docs/01-decisions/adr/impl-0001-fortigate-firewall.md) — FortiGate implementation ADR
- Core ADR-0141: Capability-Driven Network Model
- Core ADR-0167: Automated Credential Lifecycle Management (key rotation policy)

---

## Principal-Level Signal

> Firewall selection is not purely a technical decision — it's an operational sustainability decision. The best firewall is the one your team can operate, maintain, and audit at 2am during an incident. Evaluate operational complexity and vendor support availability in your geography before evaluating feature sheets.
