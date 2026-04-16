# ADR-011: SLO-Driven Alerting Model

**Status:** Accepted  
**Date:** 2026-04-16  
**Source:** IMPL-0025 (SLO Operating Model), IMPL-0026 (Alerting Strategy)  
**Project:** Sinai University Platform

---

## Context

Platform observability requires alerting. The default approach — alert on raw metric thresholds (CPU > 80%, memory > 90%) — produces high alert volume, low signal quality, and alert fatigue. Engineers stop trusting alerts. Critical incidents are missed because they are buried in noise.

IMPL-0025 and IMPL-0026 define an alternative model: alerts fire on SLO burn rate signals, not on raw metrics. An SLO burn rate alert means "the service is consuming its error budget faster than sustainable" — which is always relevant. A CPU > 80% alert may mean nothing if the service is healthy.

At Phase 0 baseline, no SLOs existed. No Alertmanager was deployed. No alert routing was defined.

## Constraints

- Team size is small — alert fatigue is a real operational risk at low headcount
- Alertmanager is the chosen routing layer (consistent with Prometheus stack decision)
- SLOs must be defined before alerting rules are written — the sequence is mandatory
- University platform has no 24/7 on-call rotation; alerts must be high-signal to be actionable

## Decision

**Define SLOs as explicit team contracts before writing any alerting rules. Alert on SLO burn rate signals, not raw metric thresholds.**

SLOs are defined per platform service (ArgoCD, Keycloak, Grafana, CoreDNS, Longhorn, ingress). Each SLO specifies: the indicator (what is measured), the objective (the target), the window (the evaluation period), and the error budget (how much failure is permitted).

Alerting rules fire when the burn rate signal indicates the error budget will be exhausted before the window closes. Two alert tiers are used: fast burn (high urgency, short window) for rapid degradation; slow burn (lower urgency, longer window) for sustained degradation below the fast-burn threshold.

Alertmanager routes alerts by severity and service. No alert is written without a corresponding SLO. Raw metric alerts (CPU, memory) are permitted only for capacity planning dashboards — they do not page.

## Alternatives Considered

**Raw metric thresholds only** — alert when CPU > X%, memory > Y%. Rejected: no relationship to user impact; high false positive rate; produces alert fatigue; does not tell the engineer whether the service is failing users.

**Alert on errors only** — fire when error rate exceeds a fixed threshold. Rejected: no context of how much this matters relative to total traffic; 10 errors in 100 requests is different from 10 errors in 10,000 requests; threshold becomes arbitrary.

**No alerting until Phase C** — defer to a later phase. Rejected: Phase B introduces production workloads. Operating production without alerting means incidents are discovered by users, not engineers.

## Trade-offs

**Accepted:** SLO definition requires upfront effort before any alert is operational. The team must agree on what "good" looks like for each service before the monitoring system can tell them when it is not good. This is 1–2 days of scoping work per service tier.

**Accepted:** SLO-based alerting is a more sophisticated operational model than threshold alerting. Engineers unfamiliar with error budgets require orientation. Mitigation: SLO runbook produced alongside each SLO definition.

**Gained:** Every alert that fires is directly interpretable as "this service is failing its users faster than our budget allows." The action is always the same: investigate the service health, not the metric value. Alert fatigue drops. Response quality improves.

## Consequences

- No alerting rule is written before the corresponding SLO document exists
- Grafana dashboards show SLO status (budget remaining, burn rate) as the primary view
- Raw metric dashboards exist for capacity planning — they do not produce pages
- Alertmanager routing is configured per service and severity
- Phase B gate includes: SLOs defined for all Tier-1 services (Keycloak, CoreDNS, ArgoCD)

## Evidence

- IMPL-0025 approved 2026-02-04 by Human Owner
- IMPL-0026 approved 2026-02-04 by Human Owner
- Phase 0 baseline confirms no SLOs, no Alertmanager, no alert routing in place

## Principal-Level Signal

Alerts are promises to the on-call engineer: "this is worth waking up for." A raw metric alert breaks that promise more often than not. An SLO burn rate alert keeps it — because it only fires when the service is failing users at a rate that matters. The discipline of defining SLOs before writing alerts forces the team to agree on what the platform is for before they define what counts as it failing. That conversation is the real value, not the alert rule.
