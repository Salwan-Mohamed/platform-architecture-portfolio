# ADR-009: Keycloak as Identity Broker — Active Directory to OIDC

**Status:** Accepted  
**Date:** 2026-04-16  
**Source:** IMPL-0023 (OIDC Authentication), IMPL-0024 (Group-Based Authorization)  
**Project:** Sinai University Platform

---

## Context

Sinai University operates Active Directory (`su.intra`, domain controllers KANSYSDC01/KANSYSDC02) as the institutional identity source. It holds all staff, faculty, and student accounts with established Kerberos/NTLM authentication. Platform services including ArgoCD, Grafana, and Alertmanager each require authentication. Without a unified strategy, each service would implement AD integration independently — producing inconsistent authentication behaviour, no single sign-on, and multiplied operational burden.

At Phase 0 baseline, no OIDC provider existed. Keycloak was not deployed. There was no SSO across any platform service.

## Constraints

- AD is the institutional source of truth and will not be replaced
- Budget excludes cloud identity services (Azure AD/Entra)
- Platform services must support OIDC natively (ArgoCD, Grafana do; custom services must)
- LDAPS (port 636, encrypted) is mandatory — plaintext LDAP on port 389 is forbidden
- No shadow identity stores — all credentials must remain in AD
- University compliance requires audit trails for platform authentication events

## Decision

**Deploy Keycloak as the single OIDC identity broker between Active Directory and all platform services.**

Keycloak is deployed at `sso.platform.su.intra` within the Kubernetes platform. It federates with Active Directory via LDAPS, translating Kerberos-held AD credentials into OIDC tokens consumed by platform services. AD groups flow through Keycloak claims into platform RBAC.

The identity chain is: `User → Platform Service → Keycloak → Active Directory (LDAPS) → Token issued → Service authorizes`.

All platform services register as OIDC confidential clients. No local Keycloak users are created. No passwords are stored in Keycloak. AD account lockout policies remain the security boundary.

**Token lifetimes:** Access tokens expire in 5 minutes. Refresh tokens expire in 30 minutes. Short lifetimes chosen deliberately — university platform security posture over user convenience.

## Alternatives Considered

**Direct AD integration per service** — each service implements LDAP independently. Rejected: no SSO, multiplied integration complexity, inconsistent behaviour across services, impossible to enforce policy centrally.

**Azure AD / Entra ID** — use Microsoft cloud identity broker. Rejected: introduces cloud dependency; AD is on-premises; adds ongoing licensing cost; violates infrastructure locality principle.

**Dex or Authentik** — alternative OIDC providers. Rejected: Keycloak has broader enterprise feature set, stronger AD federation support, larger operational community.

**SAML 2.0 instead of OIDC** — older federation protocol. Rejected: OIDC has better native support in Kubernetes-adjacent tooling; ArgoCD and Grafana both prefer OIDC; SAML complexity without benefit.

## Trade-offs

**Accepted:** Keycloak becomes a critical dependency. If Keycloak is unavailable, no platform service authentication is possible. Mitigation: HA deployment in Phase B (multi-replica, persistent session store).

**Accepted:** AD service account management overhead. A read-only service account is required for the LDAP bind. Certificate management is required for LDAPS TLS. Both are operationally bounded and documented.

**Gained:** Every platform service authentication event is auditable through Keycloak. SSO works across all services from day one of each service's deployment. New platform services inherit authentication by registering one OIDC client — not by re-implementing AD integration.

## Consequences

- ArgoCD, Grafana, Alertmanager, and all future platform services authenticate through Keycloak OIDC
- `sso.platform.su.intra` becomes a Tier-1 platform service — SLO required, HA required in Phase B
- AD group membership changes propagate to platform RBAC without any platform-side intervention
- Platform authentication is auditable by default — Keycloak emits authentication events
- Local admin backdoors to platform services are prohibited by this ADR

## Evidence

- IMPL-0023 approved 2026-02-04 by Human Owner
- IMPL-0024 (group-based authorization) builds directly on this decision
- Phase 0 baseline confirms: Keycloak not present, no OIDC, no SSO (INITIAL-PLATFORM-ABSENCE-STATEMENT.md)
- AD DCs (KANSYSDC01/02) confirmed operational and federated in Phase B planning

## Principal-Level Signal

Identity architecture is not an implementation detail. The moment you allow each service to implement authentication independently, you have created a governance problem that compounds with every new service. Keycloak centralises not just the authentication mechanism but the policy enforcement point, the audit log, and the group-to-role mapping. That is the Principal-level decision: one identity boundary, not N service-specific integrations.
