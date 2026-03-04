# Case Study 03: Multi-Site Network Architecture for Sinai University

**Type:** Network Architecture + Documentation System Design  
**Scale:** 3 campuses, 34 interfaces, 67 firewall policies, 5 VPN tunnels  
**Duration:** 2025 – ongoing  
**Role:** Network Architect + Documentation Owner  
**Key Repos:** `university-network-architecture`, `network-automation-aruba`

---

## The Problem

Sinai University's network existed. It worked, mostly. But it had no authoritative documentation. Engineers knew the topology because they built it — not because it was written down. The first time a new engineer needed to troubleshoot a VPN failure, they were starting from zero.

This is the most dangerous state for infrastructure: *operational but undocumented*. It means the organization's institutional knowledge lives in people, not systems. People leave. Memories fade. Configs drift.

---

## The Environment

| Component | Details |
|-----------|--------|
| Site: Kantra | FortiGate-1101E v7.6.4, 9 active interfaces, 36 routes, 67 firewall policies |
| Site: Katamia | FortiGate edge firewall, template documented, awaiting data collection |
| Site: Arish | FortiGate edge firewall, template documented, awaiting data collection |
| Datacenter | VPN role TBD, template structure defined |
| VPN Architecture | Full-mesh IPsec: Kantra↔Arish (✅), Kantra↔Katamia (⚠️), Arish↔Katamia (planned) |
| Campus switching | Aruba switches managed via AWX automation |
| Security zones | WAN, LAN, DMZ-Server, DMZ-Voice, VPN, Management |

---

## The Constraints

1. **Design-first requirement**: Document the architecture before optimizing it. No recommendations until as-is state is fully captured.
2. **No recommendations in the as-is repository**: As-is is for recording reality, not improving it. Mixing the two corrupts both.
3. **Completeness over selectivity**: Document everything, including unused interfaces and disabled policies. What's disabled today is often re-enabled tomorrow.
4. **Accuracy over memory**: All documentation must come from actual device output, not engineer recall.
5. **Maintainability**: One engineer must be able to keep this current without significant overhead.

---

## The Architecture Decision: Design-First Documentation

The most important architectural decision was methodological, not technical: **design-first, as-is second, never mixed.**

```
repository structure:
  branches/[site]/[site]-design.md    ← WHY the network is designed this way
  configs/[site]/                     ← WHAT the network actually looks like now

Never conflate the two.
```

Why this matters: When you mix design intent with as-is state, you lose both. The design section gets cluttered with operational notes. The as-is section gets speculative improvements. Neither is trustworthy as a single source of truth.

---

## The Critical Finding: Cairo-S2S VPN Failure

Documented finding at Kantra-site:

| Tunnel | State | Details |
|--------|-------|---------|
| Arish_S2S | ✅ Up | IPsec Phase 1 + Phase 2 operational, traffic passing |
| SU_Remote | ✅ Up | Remote access VPN operational |
| Cairo-S2S | ⚠️ Failed | IKE Phase 1 established, **1,135 IPsec Phase 2 attempts all failed** |

This finding was only discoverable because as-is documentation was collected systematically. Without the documentation, this tunnel failure might have remained undiscovered until a user reported connectivity loss — by which point the blast radius would be larger.

**Architecture lesson:** The documentation system's value is most visible at failure time, not at steady state.

---

## Network Topology

```
                    FULL-MESH IPsec VPN
              (No datacenter hub dependency)

        ┌───────────────┐     
        │  DATACENTER   │      Each site has:
        │  (Role TBD)   │      - Local internet breakout
        └───────────────┘      - FortiGate edge firewall
                                - Direct VPN to other branches
                                - Independent operation if DC down

    KANTRA-SITE ◄─────────► KATAMIA-SITE
    (FortiGate-1101E)   |       (FortiGate)
    9 active interfaces  |       Template documented
    67 policies          |       Data collection pending
    5 VPN tunnels        |
         ▲               |
         │ IPsec          |
         ▼               ▼
    ARISH-SITE ◄───────── (planned)
    (FortiGate)
    Template documented
    Data collection pending

Security zones per site:
  WAN │ LAN │ DMZ-Server │ DMZ-Voice │ VPN │ Management
```

---

## Network Automation: Aruba Campus Switching

Parallel to the FortiGate work, the campus Aruba switching layer was automated via Ansible + AWX:

- `network-automation-aruba`: Automated Aruba switch configuration deployment
- `aruba_stack_N`: Stacked switch configuration management
- `Aruba-switch-ansible_wax-automation`: AWX-integrated automation workflows

Key decision: **automation generates configuration but doesn't bypass documentation.** Every automated change produces a commit in the configuration repository. The automation serves the documentation system, not the other way around.

---

## The Trade-offs

| What I gave up | What I gained |
|----------------|---------------|
| Incomplete documentation (faster to publish) | Trustworthy documentation (accurate to device state) |
| Mixing as-is with recommendations | Clean separation between reality and intent |
| Single monolithic network document | Modular per-site documentation that scales |

---

## Outcome

- **Kantra-site: fully documented** — 34 interfaces, 36 routes, 67 policies, 5 VPN tunnels as of December 28, 2025
- **3 branch design documents** created — architectural intent for each site
- **Cairo-S2S VPN failure discovered** through systematic documentation — investigation underway
- **Standardized templates** for all sites — any engineer can collect and populate data for Katamia and Arish
- **Automation** for campus switching reduces error rate and creates audit trail for switch changes

---

## Operational Lessons

### Lesson 1: As-is documentation before optimization
Never optimize a network you haven't fully documented. The optimization will fix the obvious problems. The documentation will find the non-obvious ones.

### Lesson 2: Design-first repos age better
The `branches/[site]/[site]-design.md` documents don't become stale as fast as the `configs/[site]/` documents. Design intent is more stable than operational state. By separating them, we preserve the design thinking even when the operational state changes.

### Lesson 3: Templates enable consistent data collection
The standardized templates for Katamia and Arish mean that when any engineer collects data from those sites, the result fits the existing structure. Without templates, each site would have different documentation formats, making cross-site comparison impossible.

---

*This case study documents the discipline of treating network documentation as a first-class architectural artifact — not an afterthought.*
