# ADR-015: Four-Component Observability Stack

**Status:** Accepted  
**Date:** 2026-04-17  
**Source:** IMPL-0007 (Prometheus), IMPL-0008 (Grafana), IMPL-0009 (Loki), IMPL-0016 (Robusta)  
**Project:** Sinai University Platform

---

## Context

Platform observability requires three signals: metrics (what is happening numerically), logs (what happened textually), and network flows (what is communicating with what). A fourth operational concern: when an alert fires, engineers need context immediately — not just the alert message but the relevant pod logs, recent events, and runbook link.

Each component of the observability stack is a separate decision. But the stack only works as a system — Prometheus metrics flow to Grafana dashboards; Loki logs are queried from the same Grafana instance; Robusta enriches Alertmanager alerts with context from both. Documenting these as independent decisions obscures the integration architecture.

Hubble (IMPL-0029 / ADR-010) provides the fifth signal — network flows — and is already documented separately as it is architecturally coupled to the CNI decision.

## Constraints

- No commercial observability spend (Datadog, Splunk, Elastic Cloud excluded)
- All components must be self-hosted on the platform clusters
- Metrics and logs must be queryable from a single interface — operators cannot context-switch between tools during an incident
- Alert routing must carry enough context for a site ops engineer to begin investigation without needing to SSH into a node first
- University data (student records, grades) must not appear in log streams — FERPA compliance requires log filtering at collection

## Decision

**Deploy a four-component observability stack: Prometheus (metrics) + Grafana (visualization) + Loki (logs) + Robusta (alert enrichment). Hubble provides the fifth signal (network flows) and is managed under ADR-010.**

**Prometheus:** Metrics backend. Deployed with the kube-prometheus-stack Helm chart, which includes node exporters, kube-state-metrics, and pre-built alerting rules. Scrapes all platform services and workload namespaces. Federation or remote write used for multi-cluster metric aggregation.

**Grafana:** Single visualization interface for metrics and logs. Prometheus and Loki configured as data sources in the same Grafana instance. AD group-based dashboard access: platform admins see all dashboards; app developers see their namespace dashboards; Grafana admin access requires `platform-admins` AD group. No local Grafana users.

**Loki:** Log aggregation backend. Promtail agents deployed as DaemonSets on all nodes, collecting container logs and forwarding to Loki. Log retention configured per log stream based on compliance requirements. Student-identifying fields are filtered at the Promtail collection stage before reaching Loki — FERPA boundary is at collection, not query.

**Robusta:** Alert enrichment. Sits between Prometheus Alertmanager and the notification channel (email, Slack). When an alert fires, Robusta automatically fetches: the relevant pod logs for the alerting workload, recent Kubernetes events, resource utilization graphs, and the runbook URL for the alert. The enriched alert is what the engineer receives — not a bare metric threshold breach.

## Alternatives Considered

**Datadog** — commercial unified observability platform. Rejected: per-host licensing cost is prohibitive at university budget. Data residency: log data leaving the university network is a compliance concern.

**Elastic Stack (ELK)** — Elasticsearch + Logstash + Kibana for logs. Rejected: Elasticsearch resource consumption at scale exceeds platform node sizing; Loki's label-indexed model is more efficient for Kubernetes log patterns; maintaining a separate Kibana alongside Grafana creates two query interfaces.

**Victoria Metrics instead of Prometheus** — drop-in compatible, more efficient at scale. Rejected: kube-prometheus-stack bundles Prometheus with pre-built Kubernetes alerting rules that Victoria Metrics does not provide out of the box. For a small team, the bundled rules are worth more than the efficiency gain at current scale.

**No alert enrichment (Robusta excluded)** — send raw Alertmanager alerts. Rejected: a bare alert contains only the metric name, value, and labels. The engineer then manually opens kubectl, fetches pod logs, checks events, finds the runbook. Robusta eliminates that procedure. For a team where the same person is architect and on-call, that elimination matters.

## Trade-offs

**Accepted:** Four components to operate and upgrade. Each component has its own Helm chart, release cadence, and configuration surface. Mitigation: all deployed via ArgoCD from GitOps repo; upgrades are a Git commit, not a kubectl command.

**Accepted:** Loki retention cost. Long log retention requires storage. Longhorn (IMPL-0022) provides the backing storage. Retention windows are configured per namespace based on data classification.

**Gained:** Single query interface for all signals. An incident investigation starts in Grafana: metrics show what degraded, Loki shows the logs from the degraded service, Robusta has already attached both to the alert that woke the engineer up. The workflow is linear, not multi-tab.

**Gained:** FERPA compliance by construction. Student-identifying data filtered at Promtail before it reaches Loki. The compliance boundary is enforced in the collection agent, not dependent on query-time access controls (which can be bypassed by direct API access).

## Consequences

- Grafana is the single observability UI — engineers use one tool for metrics and logs
- All alerts are enriched by Robusta before delivery; bare Alertmanager alerts are not sent to on-call channels
- Promtail DaemonSet runs on every node — log collection has no gaps
- SLO dashboards (ADR-011) are implemented in Grafana using Prometheus and Loki data sources
- Hubble dashboards (ADR-010) are added to the same Grafana instance — network flows queryable from the same UI
- Log retention policy is documented as a governance artifact, not an operational habit

## Evidence

- IMPL-0007, -0008, -0009, -0016 accepted Phase A
- IMPL-0025 (SLO operating model / ADR-011) explicitly references Prometheus as the SLO metric source
- IMPL-0026 (alerting strategy / ADR-011) explicitly references Alertmanager + Robusta for enrichment
- FERPA compliance requirement confirmed in Sinai University platform context

## Principal-Level Signal

Observability is not a list of tools. It is a decision about what questions engineers can answer during an incident without needing to log into a node. The four-component stack answers: what degraded (Prometheus), what the service said when it degraded (Loki), what was communicating with it (Hubble), and what context the engineer needs to act (Robusta). Choosing tools without answering those questions first produces a monitoring dashboard that nobody opens during incidents.
