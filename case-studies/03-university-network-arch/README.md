# Case Study 03 — University Multi-Site Network Architecture

> *Designing full-mesh IPsec VPN for a 3-branch university with zero hub dependency*

**Repository**: [university-network-architecture](https://github.com/Salwan-Mohamed/university-network-architecture) (private)  
**Status**: Kantra-site fully documented · Katamia + Arish templates ready  
**Scale**: 3 branches · 5 VPN tunnels · 34 interfaces · 67 firewall policies

---

## Problem

Sinai University operated three geographically separated branches with inter-branch connectivity dependent on a central datacenter. When the datacenter had issues, branches lost communication with each other. There was no as-is documentation of the network — configuration knowledge lived entirely in individual engineers' heads.

Two simultaneous problems:
1. **Architectural**: Hub-dependent topology created a single point of failure
2. **Documentation**: No organization-owned record of what the network actually was

---

## Environment

- **Edge firewalls**: FortiGate-1101E at each branch (FortiOS v7.6.4)
- **Connectivity**: Each branch has independent internet breakout
- **VPN**: IPsec site-to-site between branches
- **Switching**: Aruba fabric switching with VLAN segmentation
- **Voice**: Cisco UC (CUCM/Unity) at primary site
- **Scale**: Kantra — 34 physical interfaces, 67 firewall policies, 36 routing entries

---

## Constraints

- **No service disruption**: University academic calendar — changes must be carefully scheduled
- **Existing hardware**: FortiGate appliances already deployed — not a greenfield design
- **Operations capability**: Changes must be operable by staff familiar with FortiOS
- **Documentation gap**: Starting from zero — no existing network documentation

---

## Architecture Decision

### Full-Mesh Over Hub-and-Spoke

The critical architectural decision: **eliminate hub dependency for inter-branch connectivity**.

```
        BEFORE (Hub-and-Spoke)
        ┌─────────────┐
        │  Datacenter │ ← Single point of failure
        │    (Hub)    │
        └──────┬──────┘
               │
       ┌───────┴────────┐
       ↓       ↓        ↓
    Kantra  Katamia   Arish

        AFTER (Full-Mesh)
    [Kantra]──────────[Katamia]
       │                  │
       └──────[Arish]──────┘

  3 direct IPsec tunnels · No hub dependency
  Each branch: local internet breakout + 2 VPN tunnels
```

**Decision driver**: With 3 branches and independent ISP connections at each site, full-mesh adds only 1 additional tunnel per site while eliminating datacenter as connectivity dependency.

### Security Zone Architecture (Per Site)

| Zone | Purpose |
|------|---------|
| WAN | Untrusted — Internet (primary + backup) |
| LAN | Trusted — Internal user networks |
| DMZ-Server | Published services (web, applications) |
| DMZ-Voice | Voice infrastructure (CUCM, Unity) |
| VPN | Encrypted site-to-site mesh tunnels |
| Management | Out-of-band device management |

### Documentation-First Approach

Design-first repository: architectural understanding documented separately from operational config:
- `branches/*/` — Architecture documents (design intent)
- `configs/*/` — As-is inventory (current operational state)

This separation prevents the common failure where documentation and reality diverge.

---

## Trade-offs

| Decision | Trade-off Made | Rationale |
|----------|----------------|----------|
| Full-mesh over hub-and-spoke | 1 more tunnel per site | Eliminates datacenter SPOF for inter-branch connectivity |
| Local internet breakout at each site | Split-tunnel complexity | Performance — backhauling all internet via datacenter unacceptable |
| FortiGate policy-based NAT | Less flexible than IP pool NAT | Matches existing FortiOS expertise in operations team |
| Design docs separate from configs | Dual maintenance surface | Prevents config drift from obscuring architectural intent |

---

## Outcome

**Kantra-site fully documented**:
- 34 interfaces cataloged (9 active, 25 available for expansion)
- 5 VPN tunnels documented (Arish ✅ active, SU_Remote ✅ active, Cairo ⚠️ IKE-only issue identified)
- 67 firewall policies documented (59 enabled, 8 disabled)
- 36 routing entries documented (7 connected, 27 static, 2 default)

**Critical finding surfaced through documentation**:
- Cairo-S2S VPN: IKE Phase 1 established but **1,135 IPsec Phase 2 attempts failed**
- This issue was **unknown before documentation** — existed silently in production
- Documentation created the visibility to surface the problem

### What I Would Do Differently

1. **Document all sites in parallel** — Kantra-only documentation creates false confidence
2. **Network topology diagram before config collection** — drawing the intended architecture first reveals gaps faster
3. **Automated config backup from Day 1** — manual collection is error-prone and creates staleness risk

---

## Operational Lessons

1. **Documentation surfaces problems that monitoring misses** — the Cairo VPN failure was invisible until documentation forced examination of every tunnel
2. **Design intent separate from operational state** — configs change; architecture documents change slower; they need separate homes
3. **FortiGate policy count is a quality indicator** — 67 policies with 8 disabled suggests policy sprawl starting; technical debt to address
4. **Full-mesh IPsec is simpler to operate than it sounds** — 3 tunnels between 3 sites; each site owns 2 tunnels; failure domain is per-tunnel, not per-hub
