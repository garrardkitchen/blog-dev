---
aliases: [/blog/hosted-agent-mcp-toolbox-fix/]
title: "From a Foundry Toolbox That Connected to an Agent That Could Use It"
date: 2026-08-02
draft: false
featured: true
tags: [ai, foundry ai, toolbox, azure, csharp, c#, agent inspector, fix]
---

I started with the [Foundry Toolbox MCP Skills sample](https://github.com/microsoft-foundry/foundry-samples/tree/main/samples/csharp/hosted-agents/agent-framework/foundry-toolbox-mcp-skills) installed through the Foundry Toolkit in VS Code. In this post, you will learn how to recognise the difference between *Agent Skills* and normal *MCP tools*, wire a Foundry Toolbox into a .NET hosted agent correctly, and prove that the model can actually see the toolbox. The benefit is simple: a successful connection becomes a usable capability rather than a reassuring—but misleading—startup message.

## The subtle mismatch

The sample was designed for Agent Skills. Its central integration was `UseMcpSkills`, which discovers resources exposed under the `skill://` scheme and supplies progressive-disclosure instructions to the model.

```csharp
var skillsProvider = new AgentSkillsProviderBuilder()
    .UseMcpSkills(mcpClient)
    .Build();

ChatOptions = new ChatOptions { ModelId = deployment },
AIContextProviders = [skillsProvider];
```

That is the right pattern when the toolbox contains **Agent Skills**. But a toolbox created through the Foundry Toolkit commonly contains regular capabilities—MCP servers, Web Search, Azure AI Search, OpenAPI, or Code Interpreter. Those capabilities are advertised through MCP `tools/list`, not as Agent Skills resources.

> [!WARNING]
> Connecting an MCP client is not the same as registering tools with the model. If the tools are not in `ChatOptions.Tools`, the model has nothing callable to select.

The first sign of this distinction was deceptive: the agent could initialise its MCP client and answer “hello,” but it had no callable toolbox functions. The startup path had succeeded; the agent/tool boundary had not.

## The working pattern: discover, register, invoke

The fix is to query the toolbox for tools and register the returned `AIFunction` instances with the Agent Framework chat options.

```csharp
await using var mcpClient = await McpClient.CreateAsync(
    new HttpClientTransport(
        new HttpClientTransportOptions
        {
            Endpoint = toolboxMcpServerUrl,
            Name = toolboxName ?? "foundry-toolbox",
            TransportMode = HttpTransportMode.StreamableHttp,
        },
        httpClient));

var toolboxTools = (await mcpClient.ListAgentToolsWithTaskSupportAsync()).ToArray();
if (toolboxTools.Length == 0)
{
    throw new InvalidOperationException("The Foundry Toolbox returned no tools.");
}

Console.WriteLine($"Discovered {toolboxTools.Length} toolbox tool(s): " +
    string.Join(", ", toolboxTools.Select(tool => tool.Name)));

ChatOptions = new ChatOptions
{
    ModelId = deployment,
    Instructions = "Use an available Foundry Toolbox tool whenever it is relevant.",
    Tools = [.. toolboxTools],
};
```

This is an Adapter pattern in practice: the MCP client adapts the Toolbox protocol into Agent Framework `AIFunction` objects, while `ChatOptions.Tools` becomes the model-facing capability contract.

> [!TIP]
> Log the discovered tool names during startup. It is the fastest way to distinguish “the endpoint is reachable” from “the model has tools it can call.”

In this case, the live toolbox returned `tool_search` and `call_tool`. Those are Tool Search meta-tools: the model first finds an appropriate tool, then invokes the selected one. Seeing them in the log is concrete evidence that the toolbox is now reachable *and* registered.

## Two small details that mattered

### 1. Send the Toolbox preview header

The current [Foundry Toolbox endpoint](https://learn.microsoft.com/en-us/azure/foundry/agents/how-to/tools/toolbox) expects the feature header on every request. Add it to the `HttpClient` used by the MCP transport:

```csharp
httpClient.DefaultRequestHeaders.Remove("Foundry-Features");
httpClient.DefaultRequestHeaders.Add("Foundry-Features", "Toolboxes=V1Preview");
```

### 2. Prefer the copied consumer endpoint

The code accepts the consumer URL copied from Foundry Toolkit as `TOOLBOX_ENDPOINT`, with `TOOLBOX_NAME` retained as a convenient fallback:

```csharp
string? toolboxEndpoint = Environment.GetEnvironmentVariable("TOOLBOX_ENDPOINT");
string? toolboxName = Environment.GetEnvironmentVariable("TOOLBOX_NAME");

Uri toolboxMcpServerUrl = !string.IsNullOrWhiteSpace(toolboxEndpoint)
    ? new Uri(toolboxEndpoint)
    : new Uri($"{projectEndpoint.TrimEnd('/')}/toolboxes/{toolboxName}/mcp?api-version=v1");
```

> [!NOTE]
> The consumer endpoint resolves the toolbox’s published default version. This lets a team update and promote a toolbox independently of the hosted agent deployment.

## One version trap

“Latest” is only useful when the service understands the protocol it sends. Upgrading the MCP SDK to 2.0 caused the Toolbox endpoint to reject the new `server/discover` method with a `400 Bad Request`. The project therefore uses the latest Toolbox-compatible MCP SDK line (`ModelContextProtocol` 1.4.1 and the aligned Agent Framework 1.15 packages), while retaining current supporting packages.

> [!CAUTION]
> Validate library upgrades against the live Toolbox, not just a successful restore. Protocol compatibility is a runtime contract.

The change is protected with unit tests for endpoint selection, required headers, empty-tool failure, and tool registration. A real startup test then confirmed that the configured toolbox returned two tools.

## Closing thought

An agent is not defined by the connections listed in its configuration. It is defined by the capabilities it can discover, understand, and use responsibly—because intelligence becomes useful only when it can cross the boundary into action with clarity and care.
