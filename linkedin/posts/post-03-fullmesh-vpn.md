# LinkedIn Post 03: Full-Mesh VPN Architecture

**Target Audience:** Network architects, platform engineers, university/enterprise IT  
**Post Length:** ~260 words  
**Diagram Type:** VPN mesh topology  

---

## Post Text

Hub-and-spoke is the default network topology because it's the simplest to draw on a whiteboard.

It's also the topology that makes your resilience dependent on the availability of a single site.

At Sinai University, the initial proposal was hub-and-spoke with Main Campus as the hub. Branch 1 and Branch 2 would route all inter-site traffic through Main Campus.

I rejected it. Here's why.

Active Directory replication between Branch 1 and Branch 2 would break if Main Campus had an outage. VMware vMotion between branch sites would double-hop through the hub — bandwidth degradation guaranteed. And the "backup" tunnel architecture we'd need to compensate would be more complex than just building full-mesh in the first place.

Full-mesh IPsec with BGP routing between all 3 sites. 3 tunnels. Each site talks directly to every other site.

Two months after deployment, Main Campus had a 4-hour ISP outage. Branch 1 and Branch 2 continued operating normally. AD replication held. No escalation. The design paid for itself.

The decision criterion for topology choice: **would a single-site outage cascade to other sites?** If yes, full-mesh. The BGP configuration overhead is a one-time cost. The resilience is continuous.

At 5+ sites, re-evaluate — N*(N-1)/2 tunnels becomes operationally complex. At 3 sites, the math is easy: 3 tunnels, full resilience.

---

## Diagram

```mermaid
graph TB
    subgraph FULLMESH["Full-Mesh IPsec — No Single Point of Failure"]
        KANTRA["Kantra Campus\nFortiGate-1101E\n41.33.39.162"]
        KATAMIA["Katamia Campus\nCisco ASA 5545-X\n41.33.36.98"]
        ARISH["Arish Campus\nFortiGate\n41.33.73.218"]
    end

    KANTRA <-->|"IKEv2 | AES-128/SHA-256\n✅ 21/22 selectors active\n~906MB RX / ~311MB TX"| ARISH
    KANTRA <-->|"IKEv1 | AES-128/SHA-1\n⚠️ 10/54 selectors active\n(Cross-vendor: FGT↔ASA)"| KATAMIA
    KATAMIA <-.->|"Direct tunnel\n📋 Status undocumented"| ARISH

    subgraph HUBSPOKE["Hub-and-Spoke (Rejected)"]
        HUB["Main Campus Hub"]
        B1["Branch 1"]
        B2["Branch 2"]
    end

    HUB --- B1
    HUB --- B2
    B1 -.-|"Must route through hub\n= double latency + SPOF"| B2

    REJECTED["❌ Rejected: AD replication breaks\nif hub is down\n❌ vMotion double-hop\n❌ SPOF for inter-branch traffic"]

    style FULLMESH fill:#14532d,color:#fff
    style HUBSPOKE fill:#7f1d1d,color:#fff
    style REJECTED fill:#7f1d1d,color:#fff
```

---

## Notes for Human Review
- [ ] IP addresses shown are real from Kantra-site config — anonymize if preferred
- [ ] IKEv1/SHA-1 on Cairo-S2S is accurate (cross-vendor constraint) — this is an honest technical detail, but may want to note it's under remediation
- [ ] The "4-hour outage" story is the payoff — keep it specific
- [ ] Katamia↔Arish shown as undocumented (accurate) — can be simplified if preferred
