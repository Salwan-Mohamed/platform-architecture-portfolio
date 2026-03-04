# ADR-004: FortiGate 1101E as Network Boundary and Security Control Plane

**Status:** Accepted  
**Date:** 2025-12-28  
**Decider:** Salwan Mohamed  
**Domain:** Security / Network  
**Project:** Nexus Platform — Sinai University Implementation

---

## Context

Sinai University has three campus sites (Kantra, Katamia, Arish) each requiring a network security boundary that: enforces traffic policy between security zones, terminates site-to-site VPN tunnels, provides UTM services (IPS, AV, web filtering), and is manageable centrally without requiring on-site access for routine operations.

The network boundary decision is foundational — it defines the trust model for the entire platform. A weak boundary means all security controls above it are potentially circumventable.

---

## Constraints

- Existing FortiGate evaluation/licensing already in place at Kantra site
- Budget constraints favor standardizing on a single vendor
- Small team — multi-vendor firewalls multiply operational overhead
- University compliance requirements for traffic inspection and logging
- Must support full-mesh IPsec VPN between all three sites
- FortiManager available for centralized management

---

## Decision

**Deploy FortiGate 1101E appliances at each campus site as the network security boundary, with full-mesh IPsec VPN between all three edge firewalls, using FortiManager for centralized policy management.**

---

## Alternatives Considered

| Option | Reason Rejected |
|--------|-----------------|
| Cisco ASA/FTD | Higher cost; separate management system; existing team skills are FortiGate-oriented |
| Palo Alto Networks | Significantly higher licensing cost for university budget |
| pfSense/OPNsense (open source) | Insufficient enterprise support; no integrated centralized management; limited UTM capabilities |
| Cloud-hosted firewall (SASE) | Latency for local site-to-site traffic; ongoing subscription cost exceeds appliance TCO at this scale; data sovereignty concerns |
| Hub-spoke VPN through datacenter | Single point of failure; adds latency for inter-branch traffic; datacenter availability impacts all branches |

---

## Trade-offs

**Given up:**
- Vendor diversity (all security eggs in FortiGate basket)
- Zero-cost option (open source firewalls)

**Gained:**
- Single skills requirement for the team — one vendor to know deeply
- FortiManager enables centralized policy management across all sites
- FortiGate's mature UTM stack covers IPS, AV, webfilter, and application control in one platform
- Full-mesh VPN means no hub dependency — branches communicate directly even if datacenter is unreachable
- 67 firewall policies at Kantra site document the exact traffic model of the platform

---

## Consequences

**Positive:**
- Kantra-site fully documented: FortiGate-1101E v7.6.4, 34 interfaces, 5 VPN tunnels, 67 policies
- Full-mesh architecture means losing one site doesn't isolate the other two
- Each branch has local internet breakout — branch availability doesn't depend on datacenter connectivity
- Policy changes via FortiManager are auditable — satisfies university change management requirements

**Negative:**
- Cairo-S2S VPN tunnel at Kantra: IKE Phase 1 up but 1,135 IPsec Phase 2 attempts failed — active investigation
- HA not configured at any site (standalone deployment) — single appliance failure = site outage
- fortilink interface not in use — FortiSwitch management not available

---

## Known Issue: Cairo-S2S VPN Failure

Documented critical finding: The Cairo-S2S tunnel at Kantra-site has IKE Phase 1 established but all 1,135 Phase 2 negotiations have failed. This is a known gap in the current operational state. Root cause under investigation. This ADR is accepted despite this finding because the architectural decision (FortiGate + full-mesh) is sound; the issue is an implementation/configuration problem, not an architecture problem.

---

## Evidence

- [university-network-architecture/configs/Kantra-site](https://github.com/Salwan-Mohamed/university-network-architecture/tree/main/configs/Kantra-site) — Full configuration inventory
- [nexus-platform-product-sinai-university IMPL-0001](https://github.com/Salwan-Mohamed/nexus-platform-product-sinai-university/blob/main/docs/01-decisions/adr/impl-0001-fortigate-firewall.md)
- 34 interfaces documented, 5 VPN tunnels, 67 firewall policies as of December 28, 2025

---

## Operational Lesson

The most important lesson from the FortiGate deployment: **document the current state before you optimize it**. The `university-network-architecture` repository was created specifically to capture as-is configuration before any redesign work. This gave us: a baseline to measure drift against, evidence for the Cairo VPN investigation, and an onboarding resource for any new network engineer who joins the team.

---

*Reference Core ADR: nexus-platform-engineering/docs/01-decisions/adr/impl-0001-fortigate-firewall.md*  
*Reference Core Capability: Firewall Control Plane*
