---
date: 2025-04-17
title: "Building a Remote MCP Server"
description: "A practical entry point for hosting MCP capabilities behind a network boundary."
tags: [ai, model-context-protocol, remote-servers, oauth, security]
draft: false
---

In this article, you'll learn where the remote MCP example lives, how remote transport changes the threat model, and what to verify first. That matters because centralising a capability also centralises the consequences of weak identity and tenant isolation.

The [sample code](https://github.com/garrardkitchen/mcp-server-example) and [walk-through](/ai/mcp-server-example/) show the mechanics of connecting a host to a remote MCP server. Treat the example as a learning scaffold; production readiness begins where the happy-path connection ends.

## What changes when the server is remote

A remote server normally uses Streamable HTTP. The client must validate the endpoint and certificate, and the server must validate the caller. Authentication proves an identity; authorisation decides which tools and resources that identity can use. Apply both at the operation and data boundary.

For a shared service:

- use HTTPS and a supported OAuth flow rather than long-lived tokens in editor settings;
- validate token issuer, audience, signature, lifetime, and scopes;
- bind every resource lookup to the authenticated tenant or subject;
- protect state-changing tools against replay and duplicate invocation;
- rate-limit expensive operations and cap response sizes;
- treat tool output and resource content as untrusted data; and
- record security-relevant calls without logging secrets or sensitive payloads.

Do not use session identifiers as authentication. Validate Origin where browser-based clients are possible, and avoid binding a development server to all network interfaces by default.

## Design for failure

Remote calls can time out after the server has completed the work. Give mutating tools idempotency semantics or a way to query status. Return structured errors that distinguish invalid input, denied access, transient dependency failure, and an internal fault. A model may choose a different next step based on that distinction, but the server remains responsible for enforcing policy.

## References
- [MCP transports](https://modelcontextprotocol.io/specification/2025-06-18/basic/transports)
- [MCP authorisation](https://modelcontextprotocol.io/specification/2025-06-18/basic/authorization)
- [VS Code MCP servers](https://code.visualstudio.com/docs/copilot/chat/mcp-servers)

## Closing thought

Remote MCP succeeds when a shared capability is easier to reach without making identity, authority, and failure harder to see.
