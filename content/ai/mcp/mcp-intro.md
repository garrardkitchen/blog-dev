---
title: "MCP and OpenAPI: Similar Shape, Different Job"
date: 2022-01-27T11:49:14+01:00
tags: [ai, model-context-protocol, openapi, agentic-systems]
---

In this article, you'll learn what the Model Context Protocol (MCP) standardises, how it differs from OpenAPI, and why agent-to-agent communication is a separate concern. That matters because useful AI integration depends as much on explicit trust and verification as it does on model capability.

MCP and OpenAPI feel related because both replace bespoke integration code with machine-readable contracts. The resemblance is useful, but only if we stop before claiming they solve the same problem.

## What MCP actually describes

MCP is a client-server protocol for connecting an AI host to context and capabilities supplied by a server. A server can expose three principal primitives:

- **Tools** are operations the model may ask to invoke. They can have side effects, so the host should keep the user in control of sensitive actions.
- **Resources** are addressable context, such as files, records, or application state.
- **Prompts** are reusable message templates offered by the server.

The protocol defines lifecycle, capability negotiation, and JSON-RPC messages. It does not define the intelligence of the model, guarantee that a tool is safe, or remove the need for authentication and authorisation. Local process transport and remote HTTP transport also create different trust boundaries.

## Where the OpenAPI comparison helps

OpenAPI describes the surface of an HTTP API: paths, operations, parameters, request bodies, responses, and security schemes. Tool definitions in MCP also use names, descriptions, and schemas, so both specifications can help software discover how to call an operation.

| Question | OpenAPI | MCP |
|---|---|---|
| Primary concern | Describing HTTP APIs | Connecting AI hosts to tools and context |
| Typical transport | HTTP | Standard I/O or Streamable HTTP |
| Core contract | Paths, methods, payloads, responses | Tools, resources, prompts, capabilities |
| Intended caller | Any API consumer | An MCP client inside an AI host |
| Side-effect control | Defined by the API and client | Tool invocation plus host/user consent policy |

An MCP server may wrap an API that already has an OpenAPI description. In that design, OpenAPI remains the service contract while MCP supplies an AI-oriented adapter: curated operations, model-facing descriptions, contextual resources, and a consent boundary. Generating MCP tools mechanically from a large API can work, but curation matters because a model needs a small, unambiguous capability surface more than it needs every endpoint.

## MCP is not an agent-to-agent protocol

MCP does not standardise autonomous agents delegating tasks to one another. A host can orchestrate several MCP servers, and an application can build multi-agent behavior above the protocol, but that orchestration is application logic. Dedicated agent-to-agent protocols address concerns such as agent discovery, task hand-off, status, and result exchange.

Keeping those layers separate makes architectures easier to reason about:

1. Use OpenAPI when you need a contract for an HTTP service.
2. Use MCP when an AI host needs governed access to tools and context.
3. Use an orchestration or agent-to-agent layer when independent agents must coordinate work.

## The security boundary

Descriptions and schemas improve interoperability; they do not establish trust. Before enabling a server, consider who operates it, which data it can read, which actions it can perform, where credentials live, and whether outputs can contain untrusted instructions. Apply least privilege, validate tool inputs, log consequential calls, and require confirmation where an operation can change external state.

## References
- [Model Context Protocol specification](https://modelcontextprotocol.io/specification/2025-06-18)
- [MCP architecture](https://modelcontextprotocol.io/docs/learn/architecture)
- [OpenAPI Specification](https://spec.openapis.org/oas/latest.html)

## Closing thought

OpenAPI and MCP become more useful together when each is allowed to describe its own boundary instead of being stretched into a universal integration language.
