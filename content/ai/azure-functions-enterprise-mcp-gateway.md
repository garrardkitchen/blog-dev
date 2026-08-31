---
aliases: [/blog/azure-functions-enterprise-mcp-gateway/]
title: "Azure Functions as Your Enterprise MCP Gateway"
description: "A practical pattern for Agentic Engineering and cloud native developers using Azure Functions to host small, governed Model Context Protocol tools for agents."
date: 2026-06-25T20:56:29+01:00
draft: false
author: "Garrard Kitchen"
tags: [ai, "azure-functions", "mcp", "azure-ai-foundry", "agentic-ai", "serverless"]
categories: ["Azure", "AI", "Development"]
summary: "Azure Functions can be a pragmatic way to expose narrow, auditable MCP tools to agents without inventing a new hosting platform. This post explains the pattern, the identity choices, and the caveats."
---

Your next agent tool might just be an Azure Function. 🚀

That sounds almost too ordinary, and that is exactly why it is interesting.

For Agentic Engineering and cloud native teams, the hard part of agentic engineering is rarely the demo. The hard part is exposing useful capabilities to agents without creating a sprawling, opaque tool surface that nobody can govern. Cloud native developers already understand Azure Functions, identity, serverless deployment, logging, and least-privilege design. The newer opportunity is to apply those familiar patterns to Model Context Protocol (MCP).

The practical thesis for this post is simple:

> Use Azure Functions to expose small, auditable, governed MCP capabilities to agents — not a giant enterprise tool belt.

> [!TIP]
> This post focuses on the enterprise pattern, not a step-by-step deployment walkthrough. Microsoft Learn already provides the detailed tutorials, and those are linked in the sources section.

## Why this matters now

Microsoft’s Azure Functions MCP extension enables Functions apps to create remote MCP servers. In plain English, that means a function app can expose capabilities that MCP clients, language models, and agents can discover and call.

The extension supports three useful shapes:

| MCP shape | What it means in a Functions-hosted server |
|---|---|
| Tool | A function can be invoked from an MCP tool call request. |
| Resource | A function can expose data or content as an MCP resource. |
| Prompt | A function can expose a prompt template or prompt-like capability. |

Microsoft’s Build 2026 update for the Azure Functions MCP extension also highlights richer capabilities such as tool, resource, and prompt triggers, MCP Apps, built-in MCP authentication, structured/rich content, and .NET fluent configuration APIs.

That is a useful direction for cloud native delivery teams because it maps agent tooling onto a platform many teams already operate.

Instead of asking, “Where should we host all these agent tools?”, a better first question is:

> Which one bounded application capability should we safely expose as an MCP tool?

## MCP in Functions in plain terms

MCP is a client-server protocol that helps agents and language models discover and use external tools and data sources. With Azure Functions, the server side can be a function app.

There are currently two Azure Functions hosting approaches to keep separate:

1. MCP extension servers
   - Use the Azure Functions MCP extension and the Functions triggers/bindings programming model.
   - Best when your team is already comfortable with Functions.

2. Self-hosted MCP SDK servers
   - Host an MCP server built with official MCP SDKs on Azure Functions using custom handlers.
   - Microsoft Learn describes this as public preview.

That distinction matters. If you blur it, you will overstate what is generally available and what is still preview.

> [!NOTE]
> For C#, Microsoft Learn states that the Azure Functions MCP extension supports the isolated worker model. The same overview states that PowerShell apps are not currently supported by the MCP extension, and Go is not currently supported for this feature. Always re-check language/runtime support before making a delivery commitment.

## The enterprise pattern: small, governed tool surfaces

The tempting mistake is to build a “company MCP server” with everything in it: project lookup, ticket updates, deployment triggers, database search, document access, incident actions, and a few mystery admin operations because they were easy to wire up.

Please do not start there.

A more defensible Agentic Engineering and cloud native pattern looks like this:

1. Pick one narrow capability.
2. Express it as one or a few MCP tools/resources/prompts.
3. Define explicit input and output contracts.
4. Decide the identity model before production use.
5. Log tool calls and outcomes.
6. Review the tool descriptions and outputs as part of your threat model.
7. Register or catalogue the server where appropriate so it can be discovered and governed.

Examples of bounded first tools might include:

- Read-only product catalogue lookup for a support agent.
- Retrieval of deployment metadata for a specific application/environment.
- A controlled summarisation endpoint over approved operational knowledge.
- A diagnostic helper that returns a small, redacted slice of build or runtime evidence.

The common thread is scope. The tool should do one useful thing, with a clear contract and boring operational behaviour.

> [!WARNING]
> An MCP endpoint is not “safe” just because it is called by an AI assistant. Tool descriptions, arguments, outputs, logs, and downstream actions all become part of the security and governance boundary.

## Transport and client choices

The Azure Functions MCP extension supports Streamable HTTP at:

```text
/runtime/webhooks/mcp
```

It also supports Server-Sent Events (SSE) at:

```text
/runtime/webhooks/mcp/sse
```

Microsoft Learn notes that newer protocol versions deprecate SSE, so you should use Streamable HTTP unless a client specifically requires SSE.

That is a good example of the sort of delivery detail that belongs in a team checklist. The protocol choice should not be an accidental copy/paste from the first sample somebody found.

## Identity is the architecture, not an afterthought

The most important production decision is not whether the demo works. It is who, or what, is allowed to call the tool — and under whose context.

Microsoft Learn describes several connection options when connecting Functions-hosted MCP servers to Microsoft Foundry Agent Service:

| Option | Useful framing |
|---|---|
| Key-based | Development-friendly and supported by default, but still a shared secret. |
| Microsoft Entra | Better for production-style identity, currently described for project managed identity in the Foundry connection guide. |
| OAuth identity passthrough | Useful when each user must authenticate and user context must persist. |
| Unauthenticated | Only for public read-only information or private development scenarios. |

There is a subtle but important point here: key-based access can make an endpoint harder to call accidentally, but it is not the same as user-aware authorization. For enterprise use, prefer Microsoft Entra or OAuth identity passthrough patterns where they fit the scenario.

> [!WARNING]
> Unauthenticated access should not be your production shortcut. Microsoft Learn describes it for development or MCP servers that access only public information. If a tool can see internal data, change state, or reveal operational context, design proper authentication and authorization from the start.

## A practical checklist for Agentic Engineering and cloud native developers

Before you roll a Functions-hosted MCP server beyond a local experiment, ask these questions.

### 1. What is the bounded use case?

If the tool cannot be explained in one sentence, it is probably too broad.

Good:

> “Given an application ID, return the current deployment version and owning team from approved metadata.”

Risky:

> “Let agents query our platform.”

### 2. What are the tool contracts?

Define the inputs, outputs, validation rules, and error behaviour. Avoid returning huge blobs of internal context because it is convenient. Agents do better with structured, relevant, minimal evidence.

### 3. What identity is being used?

Decide whether the call represents:

- The agent service.
- The project or platform identity.
- The end user via OAuth identity passthrough.
- A development-only key.

This decision affects authorization, auditing, incident response, and user trust.

### 4. What should never be returned?

Be explicit about redaction. Avoid returning secrets, tokens, unnecessary personal data, internal-only operational details, or broad environment dumps.

### 5. How will calls be logged and reviewed?

At minimum, capture enough telemetry to answer:

- Which tool was called?
- Who or what called it?
- What high-level input was used?
- What decision or response was produced?
- Was anything blocked, redacted, or failed?

### 6. Where is the server catalogued?

Microsoft Learn suggests registering MCP servers hosted in Azure Functions in Azure API Center to maintain a private organizational tool catalogue. For larger organisations, discoverability and inventory matter. A private endpoint passed around in chat is not a governance model.

### 7. What is still preview or language-specific?

Call out preview status and language/runtime support in your design notes. The Functions MCP landscape is moving, and delivery teams should not accidentally promise production support based on a preview-only path.

## A simple starter architecture

A sensible first implementation might look like this:

```text
MCP client / Foundry agent / VS Code
        |
        | Streamable HTTP
        v
Azure Functions MCP endpoint
        |
        | Validated tool trigger
        v
Small application capability
        |
        | Least-privilege access
        v
Approved data source or service
```

Then wrap that with the boring things that make it enterprise-ready:

- Microsoft Entra or OAuth where appropriate.
- Managed identity for downstream Azure access.
- Application Insights or equivalent operational telemetry.
- Secret-free client configuration.
- Code review for tool descriptions and outputs.
- Threat modelling for prompt/tool poisoning and data exfiltration paths.

It is not glamorous. That is the point. Good enterprise agent tooling should look a lot like good enterprise integration tooling.

## Caveats and trade-offs

There are several places to be careful.

First, do not treat MCP extension-based servers and self-hosted SDK-based servers as the same thing. They are related hosting options, but they have different mechanics and support statements.

Second, do not assume every feature applies equally across languages. The Azure Functions MCP documentation has language-specific notes, and some enhancements called out in blog updates may be .NET-focused.

Third, do not confuse “agent can call it” with “agent should call it.” Human approval, review gates, and normal engineering controls still matter — especially for tools that change state.

Fourth, do not put secrets into checked-in MCP client configuration. Microsoft’s example uses VS Code input prompts so secrets such as the Functions system key are not stored in the `mcp.json` file.

Finally, remember that preview features can change. Treat them as candidates for pilots and learning, not as blanket production commitments.

## What I would pilot first

For an Agentic Engineering or cloud native team, I would start with a read-only tool.

Pick one service where developers regularly ask the same operational question, such as:

- “Which version is deployed to test?”
- “Who owns this integration?”
- “What approved runbook applies to this alert?”
- “Which feature flags are enabled for this environment?”

Expose only that answer through a Functions-hosted MCP tool. Use Streamable HTTP. Avoid anonymous production access. Keep the output small. Log calls. Review the tool contract like you would review an API.

If that proves useful, add the next narrow tool. If it does not prove useful, you have learned cheaply without creating an enterprise-wide agent gateway nobody trusts.

## Conclusion

Azure Functions is not interesting here because it makes MCP magical. It is interesting because it makes MCP boring in the best possible way.

The winning enterprise pattern is not “agents everywhere.” It is safe, narrow, auditable capabilities exposed to agents through platforms and controls that application teams already understand.

For Agentic Engineering and cloud native developers, that is the real opportunity: take the serverless engineering habits you already trust, and use them to build agent tools that are useful enough to adopt and constrained enough to govern.

## Sources

- [Azure SDK Blog: Azure Functions MCP Extension: What’s New at Build 2026](https://devblogs.microsoft.com/azure-sdk/functions-mcp-updates-build-2026/)
- [Microsoft Learn: Model Context Protocol bindings for Azure Functions overview](https://learn.microsoft.com/en-us/azure/azure-functions/functions-bindings-mcp)
- [Microsoft Learn: Tutorial: Host an MCP server on Azure Functions](https://learn.microsoft.com/azure/azure-functions/functions-mcp-tutorial)
- [Microsoft Learn: Self-hosted remote MCP server on Azure Functions (public preview)](https://learn.microsoft.com/azure/azure-functions/self-hosted-mcp-servers)
- [Microsoft Learn: Connect an MCP server on Azure Functions to a Foundry Agent Service agent](https://learn.microsoft.com/azure/azure-functions/functions-mcp-foundry-tools)

Enjoy! Generated by AI Garrard (blogger) - google/gemma-4-e4b
