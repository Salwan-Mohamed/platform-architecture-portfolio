# Case Study 03: Multi-Site Network Architecture for a University

> **Role:** Network Architect + Platform Engineer  
> **Environment:** Sinai University — 3-site campus network  
> **Duration:** 2025 Q3 → 2025 Q4  
> **Complexity:** ⭐⭐⭐⭐  

---

## The Problem

A university with 3 campus sites and ~5,800 users was operating on a network that had grown organically over 10 years without a design document. VLANs had been added ad-hoc. Inter-site connectivity was a single IPsec tunnel with a pre-shared key that had never been rotated. The core routing was static. Network changes required SSH access to 30+ switches and were made by whoever happened to be on-call.

Two specific incidents made the redesign urgent:
1. A misconfigured ACL on a distribution switch blocked student Wi-Fi across an entire campus for 6 hours — with no way to identify the change or roll it back
2. A security audit found that the research network (handling data under HIPAA-adjacent requirements) was on the same VLAN as the student network in two buildings

The requirement: redesign the network to be secure, documented, automated, and resilient — without replacing hardware.

---

## The Environment

| Dimension | Detail |
|-----------|--------|
| **Sites** | Main Campus + 2 branch campuses |
| **Users** | ~5,800 (students + staff + research) |
| **Switching** | Aruba switches (HPE) — existing hardware |
| **Firewalls** | FortiGate 1101E — new deployment |
| **Segments** | Student, Staff, Research, IoT, Management, Platform |
| **Inter-site** | IPsec VPN — redesigned from single to full-mesh |
| **Management plane** | FortiManager + Aruba Central |

---

## The Constraints

- **No hardware replacement budget:** All Aruba switches remain; new topology must be achievable via configuration
- **Zero-downtime migration:** University cannot tolerate a scheduled network blackout during term
- **5 network segments required:** Student, Staff, Research, Platform/Management, IoT — each with different security postures
- **Research network compliance:** Must be isolated with IPS inspection on all ingress/egress
- **Automation requirement:** All switch configurations must be manageable via Ansible without SSH ad-hoc
- **Documentation-first:** Architecture must be documented before any change is made to production

---

## The Architecture

### Design Philosophy: Topology Defines Policy

Rather than applying security as an overlay (ACLs added to an organic topology), the redesign was topology-first: every security boundary is expressed as a physical or logical segment enforced by the firewall, not by switch ACLs.

### Network Segmentation Model

```
┌─────────────────────────────────────────────────────────┐
│                  FortiGate 1101E (per site)             │
│                 Security Policy Enforcement             │
├────────────┬─────────────┬───────────┬──────────────────┤
│  Student   │    Staff    │ Research  │    Platform      │
│  VLAN 100  │  VLAN 200   │ VLAN 300  │   VLAN 400+      │
│  /20       │   /22       │  /23      │    /24           │
│            │             │           │                  │
│  Internet  │ Internet +  │ Internet  │  Management      │
│  only      │ Internal    │ + IPS     │  only            │
└────────────┴─────────────┴───────────┴──────────────────┘
                         │
              Inter-site IPsec VPN
                   (Full Mesh)
              ┌────────────────────┐
              │  Main ↔ Branch 1  │
              │  Main ↔ Branch 2  │
              │  Branch 1 ↔ B2    │
              └────────────────────┘
                 BGP over IPsec
              Dynamic route failover
```

### Automation Model

All switch configurations managed via Ansible AWX:
- `network-automation-aruba`: Aruba switch configuration playbooks
- `network_automation_awx`: AWX inventory and job templates
- Every VLAN, trunk, and access port defined as code
- Drift detection: AWX runs configuration compliance checks on schedule

---

## The Key Decision Points

### Decision 1: Firewall-as-Policy-Enforcer, Not Switch ACLs
The previous design used switch ACLs for inter-VLAN security. This was replaced by routing all inter-segment traffic through FortiGate, which provides application-layer inspection, IPS, and centralized logging.

**Trade-off:** Traffic that previously stayed on the switching fabric now traverses the firewall. For very high-volume intra-campus traffic, this adds a chokepoint. Validated acceptable at current traffic volumes; firewall throughput headroom is sufficient.

### Decision 2: BGP over IPsec for Dynamic Routing
Static routes were replaced by eBGP peering over IPsec tunnels between sites. This enables automatic route failover when a tunnel degrades.

**Trade-off:** BGP configuration is more complex than static routes. Required a week of ops team training. Paid for itself immediately when a tunnel flap during ISP maintenance caused automatic rerouting rather than a manual intervention.

### Decision 3: Zero-Downtime Migration via VLAN Coexistence
New VLANs were provisioned in parallel with existing ones. Users were migrated in cohorts (building by building) over 4 weeks, with the old VLANs decommissioned only after 2 weeks of validation per cohort.

**Trade-off:** Required maintaining two parallel configurations simultaneously, which doubled the risk surface during migration. Mitigated by detailed change records and a rollback procedure tested in the lab before production.

---

## The Outcome

- **5 clean network segments** deployed across all 3 sites
- **Research network isolation** validated by third-party security review — compliance satisfied
- **Full-mesh IPsec** with BGP routing operational; failover tested and confirmed
- **100% switch configuration in Git** via Ansible AWX; ad-hoc SSH changes blocked by policy
- **Zero unplanned outages** during the 4-week migration window
- **Ops team automation proficiency** improved: team members can now modify switch configs via PR

---

## Operational Lessons

**1. Document the topology before touching production.**  
The university-network-architecture repository was created and filled with the target design before a single switch command was run. This enforced design discipline and gave leadership a clear picture of what was being built.

**2. Migration is a governance event, not just a technical event.**  
Each cohort migration required a change record, a rollback procedure, a test plan, and a success criterion. The ops team initially found this overhead frustrating; they changed their view after the first rollback event went smoothly because the procedure existed.

**3. BGP is overkill for 3 sites and worth it anyway.**  
The complexity cost of BGP over static routing is real. The operational benefit (automatic failover, visible route table per site) makes the static route alternative look fragile in retrospect.

---

## Source Repositories

| Repository | Role |
|------------|------|
| [university-network-architecture](https://github.com/Salwan-Mohamed/university-network-architecture) | Network architecture documentation |
| [network-automation-aruba](https://github.com/Salwan-Mohamed/network-automation-aruba) | Aruba switch automation |
| [network_automation_awx](https://github.com/Salwan-Mohamed/network_automation_awx) | AWX job templates and inventory |
| [cisco-voice-cucm-automation](https://github.com/Salwan-Mohamed/cisco-voice-cucm-automation) | Voice infrastructure automation |
