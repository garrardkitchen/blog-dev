---
title: "Using .NET Aspire to Tighten the Inner Loop"
date: 2024-07-09T08:33:48+01:00
tags: [engineering, inner-loop, dotnet-aspire, developer-experience, observability]
draft: true
---

In this article, you'll learn how .NET Aspire can make distributed application dependencies observable and repeatable during local development. That matters because orchestration is valuable only when it shortens diagnosis rather than hiding how the system is assembled.

## What the AppHost contributes

An Aspire AppHost is executable orchestration code for the application model. It declares projects, containers, executables, and backing resources, then expresses references and dependencies between them. The developer runs the AppHost to start the graph and receives a dashboard for resource state, logs, traces, and metrics.

This is not a production scheduler and it does not turn a distributed system into one process. It gives the inner loop a consistent description of the system and injects connection information through configuration. Deployment tooling can translate the model for a target environment, but production topology, scaling, identity, networking, backup, and policy still require explicit decisions.

## Model relationships, not startup scripts

A useful AppHost describes intent:

~~~csharp
var builder = DistributedApplication.CreateBuilder(args);

var cache = builder.AddRedis("cache");

builder.AddProject<Projects.Web>("web")
    .WithReference(cache)
    .WaitFor(cache);

builder.Build().Run();
~~~

WithReference makes connection information available to the consuming project. WaitFor expresses startup ordering based on resource readiness. The application should still tolerate transient failure; startup order is not a resilience strategy.

Keep secrets out of source control. Use parameters, developer secrets, or the credential mechanisms of the target platform. Treat container images and resource versions as dependencies that need pinning and maintenance.

## Use telemetry as part of development

Aspire's service defaults can configure OpenTelemetry, health checks, service discovery, and resilient HTTP behavior for .NET projects. The dashboard then makes a cross-service request trace available while the change is still in the editor. That is the real inner-loop gain: not merely starting several processes, but seeing causality across them.

Avoid adding resources solely to make the dashboard look complete. Model the smallest graph that reproduces the behavior under investigation, and use contract or integration tests for repeatable assertions.

## References
- [.NET Aspire documentation](https://learn.microsoft.com/dotnet/aspire/)
- [.NET Aspire AppHost overview](https://learn.microsoft.com/dotnet/aspire/fundamentals/app-host-overview)
- [.NET Aspire service defaults](https://learn.microsoft.com/dotnet/aspire/fundamentals/service-defaults)

## Closing thought

The best Aspire AppHost does not disguise a distributed system as a simple one; it makes the system's real relationships inexpensive enough to inspect every day.
