---
aliases: [/blog/observability/]
title: "Observability Is the Ability to Ask New Questions"
date: 2020-01-02T12:49:14+01:00
draft: true
featured: false
tags: [engineering, observability, opentelemetry, logging, metrics, tracing]
---

In this article, you'll learn how telemetry supports investigation, how to design signals around decisions, and why observability is more than collecting logs, metrics, and traces. That matters because a dashboard can be green while a customer is still unable to complete the work that matters.

Observability is a property of a system: the degree to which its internal behavior can be understood from the evidence it emits. Logs, metrics, and traces are useful signals, but installing three products does not create that property automatically.

## Begin with questions

Start with the questions operators and product teams must answer:

- Which user journeys are failing, for whom, and since which change?
- Is latency in our code, a dependency, a queue, or resource contention?
- Is an alert evidence of customer harm or only of unusual internal behavior?
- Can we distinguish a retrying request from several independent requests?
- What capacity or reliability limit are we approaching?

Instrument stable boundaries—HTTP requests, messages, scheduled jobs, database calls, and important domain transitions—with consistent service, environment, version, and correlation attributes. Prefer OpenTelemetry conventions where they fit so signals can be joined and backends can be changed with less reinstrumentation.

## Use each signal for its strength

Metrics are efficient for trends, ratios, service-level indicators, and alerting. Logs preserve discrete context and explanations. Traces show causal paths across components. Profiles reveal where running code spends time. Events and deployment metadata explain when the system changed.

Cardinality is a design constraint. A user ID or unbounded URL in a metric label can make the system expensive or unusable; that detail may belong in a sampled trace or protected log instead. Sampling also changes what can be concluded. Head sampling is inexpensive but can discard rare failures; tail sampling preserves selected outcomes at additional operational cost.

## Design the operating model

Define ownership, retention, access, and cost before data volume grows. Observability stores often contain identifiers, payload fragments, topology, and error detail, making them sensitive systems in their own right. Redact secrets at source and avoid collecting personal data without a purpose and retention policy.

Alerts should connect to a user or system objective, contain enough context to begin diagnosis, and point to a runbook. Test telemetry and alerts during normal delivery. If a trace disappears at a queue or a dashboard cannot separate versions, discover that before an incident.

A single backend can simplify correlation, while separate platforms can limit blast radius or satisfy data constraints. Choose deliberately and keep a minimal independent view of the observability platform itself; losing telemetry during an outage is a predictable failure mode.

## References
- [OpenTelemetry documentation](https://opentelemetry.io/docs/)
- [Google SRE: Monitoring distributed systems](https://sre.google/sre-book/monitoring-distributed-systems/)
- [CNCF Observability Technical Advisory Group](https://github.com/cncf/tag-observability)

## Closing thought

Telemetry tells us what a system remembered; observability depends on whether we taught it to remember the things we had not yet thought to ask.
