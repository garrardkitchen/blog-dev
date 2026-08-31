---
aliases: [/blog/azure-devops-artifacts-upstream-source-oddness/]
title: "Understanding Azure Artifacts Upstream Sources"
date: 2022-09-30T10:44:20+01:00
tags: [cloud, azure-devops, artifacts, nuget, package-management]
draft: true
---

In this article, you'll learn how Azure Artifacts upstream sources resolve and save packages, why two developers can observe different restore behavior, and how to diagnose the feed without bypassing it. That matters because package provenance is part of the software supply chain, not merely a restore convenience.

The original incident looked simple: nuget.org was configured as an upstream source, yet dotnet restore could not obtain a public package through the Azure Artifacts feed. Adding nuget.org directly made the restore succeed, but it also bypassed the feed policy we were trying to test.

## How upstream resolution works

An Azure Artifacts feed can expose packages from its own store and from configured upstream sources. When an authorised request resolves an upstream package, Azure Artifacts can save that version into the feed. Later consumers receive the saved copy even if the public source changes or is unavailable.

That behavior improves repeatability, but it introduces policy and permission questions:

- Is nuget.org enabled and ordered as expected?
- Can the requesting identity read the feed?
- Does the identity responsible for first-time upstream saving have the required role?
- Is a package-source mapping or nuget.config excluding the intended source?
- Is the requested version already saved, hidden by a view, or blocked by policy?
- Are credentials being loaded from the project, user profile, credential provider, or CI task?

Use dotnet nuget list source and dotnet restore with diagnostic verbosity, then redact credentials before sharing the log. Inspect every nuget.config in the documented hierarchy; configuration files are combined, and a clear element can remove sources inherited from a higher level.

## Avoid the misleading fix

Adding nuget.org directly is a useful isolation test. It proves the package and network path are available, but it does not repair the upstream configuration. Leaving the direct source in place can allow public packages to bypass feed retention, provenance, allow-listing, or dependency-confusion controls.

Restore through one intentional configuration. In CI, authenticate using the supported Azure Artifacts credential mechanism, scope tokens minimally, and make source mapping explicit where private package names could collide with public ones. Pin package versions and treat a successful restore as the start of provenance verification, not its conclusion.

## A diagnostic sequence

1. Reproduce with a clean package cache and the exact CI nuget.config.
2. Confirm source order and mapping.
3. Authenticate as the failing identity.
4. Check feed and upstream status, views, and permissions.
5. Request one known public package version through the feed endpoint.
6. Inspect whether the version is saved after the request.
7. Compare a direct nuget.org restore only as a controlled test.
8. Record the cause before changing policy.

## References
- [Azure Artifacts upstream sources](https://learn.microsoft.com/azure/devops/artifacts/concepts/upstream-sources)
- [Upstream source behavior](https://learn.microsoft.com/azure/devops/artifacts/how-to/set-up-upstream-sources)
- [NuGet configuration file reference](https://learn.microsoft.com/nuget/reference/nuget-config-file)

## Closing thought

A package feed is where convenience becomes governance: if we cannot explain why a dependency arrived, we do not yet control how software enters the system.
