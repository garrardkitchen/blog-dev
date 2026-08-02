---
aliases: [/blog/hosted-agent-template-fix/]
title: "Making a Foundry Hosted-Agent Template Work Locally in Agent Inspector"
date: 2026-08-02
draft: false
featured: true
tags: [ai, foundry ai, azure, csharp, c#, agent inspector, fix]
---

This post shows how to take the C# **MCP Tools** hosted-agent template installed by Foundry Toolkit and make it work both in VS Code's local Agent Inspector and in the Foundry hosted playground. You will learn why two identity-related errors appear only on a developer machine, and how to fix them without weakening the production security model.

The starting point was the [MCP Tools sample](https://github.com/microsoft-foundry/foundry-samples/tree/22b2c89c676bddb107ea370330d6341e25ff674b/samples/csharp/hosted-agents/agent-framework/mcp-tools), installed through Foundry Toolkit in VS Code. The same code ran correctly after deployment but failed locally. That difference is the clue: local tools simulate an agent endpoint; they do not recreate every capability of the Foundry platform.

> [!TIP]
> When code works in the hosted playground but not locally, compare **platform-provided context** first: identity headers, managed identity, injected environment variables, and network access.

## Problem one: Agent Inspector is local, not the Foundry gateway

The first request failed with this error:

```text
The registered HostedSessionIsolationKeyProvider returned null for the current request.
Ensure the Foundry platform is providing the x-agent-user-id header...
```

`x-agent-user-id` is a trusted platform header. Foundry injects it into a hosted container after identifying the caller and uses it to partition per-user session state. By design, VS Code's Agent Inspector connects directly to the local agent endpoint; it is not a proxy for the deployed Foundry service and therefore cannot create that platform identity. [Microsoft's Agent Inspector documentation](https://learn.microsoft.com/en-us/azure/foundry/agents/how-to/agent-inspector) and the [hosted-agent runtime contract](https://learn.microsoft.com/en-us/azure/foundry/agents/concepts/hosted-agent-contract) make that boundary explicit.

Sending `x-agent-user-id: local-inspector` with `curl` is useful to prove the diagnosis, but it is not the right long-term fix. The header is an inbound value, and allowing arbitrary callers to establish a production identity would defeat session isolation.

> [!WARNING]
> Never add middleware that fabricates `x-agent-user-id` for a deployed agent. Only Foundry should establish that user identity in production.

Instead, register a fallback provider **only when the process is not hosted by Foundry**:

```csharp
var isFoundryHosted = FoundryEnvironment.IsHosted;

var builder = AgentHost.CreateBuilder(args);

if (!isFoundryHosted)
{
    builder.Services.AddSingleton<
        HostedSessionIsolationKeyProvider,
        LocalSessionIsolationKeyProvider>();
}

builder.Services.AddFoundryResponses(agent);
```

The provider supplies one stable identity for local Inspector sessions:

```csharp
internal sealed class LocalSessionIsolationKeyProvider : HostedSessionIsolationKeyProvider
{
    private const string LocalInspectorUserId = "local-inspector";

    public override ValueTask<HostedSessionContext?> GetKeysAsync(
        ResponseContext context,
        CreateResponse request,
        CancellationToken cancellationToken) =>
        ValueTask.FromResult<HostedSessionContext?>(
            new HostedSessionContext(LocalInspectorUserId));
}
```

In Foundry, this registration is skipped. The SDK's production provider continues to use the platform-injected identity. The customisation API is currently marked experimental by the package, so keep the warning suppression narrow and revisit it when upgrading the hosting package.

> [!NOTE]
> The local fallback deliberately represents one development user: `local-inspector`. It is a development convenience, not an identity or authentication mechanism.

## Problem two: `DefaultAzureCredential` tried an Azure VM identity on a Mac

After adding the request header during diagnosis, the next failure was:

```text
ManagedIdentityCredential authentication failed...
No route to host (169.254.169.254:80)
```

That IP address is Azure's Instance Metadata Service (IMDS). It is available to Azure-hosted workloads, not a local macOS process. The Azure CLI login was valid, but this local IMDS probe failed before the credential chain reached the CLI identity.

> [!IMPORTANT]
> Do not replace `DefaultAzureCredential` with a hard-coded Azure CLI credential. `DefaultAzureCredential` is still the right abstraction; only its local managed-identity probe needs to be disabled.

Keep `DefaultAzureCredential`; it is still the right cloud-native pattern. Configure it from the same deployment-context decision instead:

```csharp
var isFoundryHosted = FoundryEnvironment.IsHosted;
var credential = new DefaultAzureCredential(
    FoundryCredentialOptions.Create(isFoundryHosted));

AIAgent agent = new AIProjectClient(projectEndpoint, credential)
    .AsAIAgent(/* model, instructions, and tools */);
```

```csharp
internal static class FoundryCredentialOptions
{
    internal static DefaultAzureCredentialOptions Create(bool isFoundryHosted) =>
        new()
        {
            ExcludeManagedIdentityCredential = !isFoundryHosted,
        };
}
```

Locally, the credential now reaches `AzureCliCredential` after `az login`. In Foundry, managed identity remains enabled and is used as intended. This is the **Strategy pattern** applied to authentication: the code selects the appropriate credential path from the runtime environment while the consuming `AIProjectClient` stays unchanged.

## The result

The template now supports an ordinary local Inspector session without manually injecting headers, while the deployed agent still relies on Foundry-provided identity and managed identity. Unit tests cover both branches: local excludes managed identity and supplies `local-inspector`; hosted leaves managed identity enabled.

> [!TIP]
> Treat the local and hosted paths as separate security contexts. Test both whenever you change authentication, session state, or user-scoped storage.

Local development is valuable precisely because it is not production. The goal is not to imitate every cloud guarantee on a laptop, but to make the differences explicit—so that convenience in development never becomes an accidental compromise in the system we ship.
