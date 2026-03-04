# ADR-004 — FortiGate as Network Control Plane

**Status**: Accepted | **Date**: 2025-12 | **Author**: Salwan Mohamed

## Context
Sinai University had existing FortiGate-1101E appliances deployed at all branch sites. The decision is not "which firewall to buy" but "whether to build the platform governance model around existing FortiGate infrastructure or replace it." The operations team has FortiOS expertise; replacement requires retraining, migration windows, and procurement cycles.

## Decision
**Retain FortiGate-1101E as the network control plane and build platform capabilities around it.**

FortiGate role: Perimeter security enforcement · VPN endpoint for full-mesh IPsec · UTM (IPS, AV, Web Filtering, App Control) · Network segmentation (WAN/LAN/DMZ/Voice/VPN zones)

**Critical Finding During Documentation**: Documenting the as-is state revealed a production issue invisible before documentation — Cairo-S2S VPN tunnel: IKE Phase 1 UP but 1,135 IPsec Phase 2 attempts FAILED. Zero traffic flowing despite configured tunnel. This validates the documentation-first approach.

## Consequences
**Positive**: No procurement/migration cost · Existing team expertise preserved · FortiGate-1101E enterprise-grade · FortiOS 7.6.4 modern security features  
**Negative**: Not GitOps-native (requires FortiOS API or FortiManager) · Policy sprawl risk (67 policies at Kantra) · Vendor lock-in

## Alternatives Considered
| Alternative | Rejected Because |
|-------------|-----------------|
| pfSense/OPNsense | No enterprise support; insufficient UTM features |
| Palo Alto | Budget; team retraining; no existing investment |
| Cisco ASA/FTD | Higher operational complexity |

## Signals
- Zero unplanned outages caused by firewall policy changes
- All policy changes traceable to change management tickets
- VPN tunnel uptime >99.5% across all branch-to-branch tunnels

## References
- Core ADR: Firewall Control Plane Capability Contract
- Product ADR: IMPL-0001
- Case study: 03-university-network-arch


# ADR-005 — Vendor-Agnostic Core Architecture

**Status**: Accepted | **Date**: 2025-12 | **Author**: Salwan Mohamed

## Context
The Core governance foundation will be consumed by multiple Product implementations with different vendor choices. If Core contains vendor-specific language ("use ArgoCD", "use FortiGate"), the Core becomes a how-to guide rather than a governance framework. Products can't differ without "violating" it.

## Decision
**The Core must contain zero vendor names, product names, or tool-specific configuration. The Core defines capability contracts; Products choose implementations.**

```
Core Language:              Product Language:
  "GitOps Reconciler"   →     "ArgoCD" / "Flux CD"
  "Kubernetes Control   →     "RKE2" / "EKS" / "GKE"
     Plane"
  "Firewall Control     →     "FortiGate" / "Palo Alto"
     Plane"
  "Identity Broker"     →     "Keycloak" / "Okta"
```

## Consequences
**Positive**: Core reusable across different technology stacks · Products can upgrade/replace vendors without Core modification · Core review focuses on governance, not tool opinions  
**Negative**: Higher abstraction less immediately actionable · Capability contract definitions require more precision

## Signals
- Core reviewed and zero vendor names found in any ADR
- Two or more products implementing same Core with different vendors
- Product vendor changes require zero Core ADR modifications

## Enforcement
- Core ADR reviews check for vendor names (rejected if found)
- Product ADRs must reference the Core capability they implement
- `core-compliance-checker.sh` validates product ADRs reference Core ADRs
