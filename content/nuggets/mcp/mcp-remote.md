---
date: 2025-04-17
title: "Remote MCP Checklist"
description: "A compact checklist for reviewing a remote MCP connection."
tags: [ai, model-context-protocol, remote-servers, checklist]
---

This checklist helps you review a remote MCP endpoint's identity, authority, transport, and data handling. That matters because a convenient shared server can otherwise become a shared escalation path.

## Checklist

- Verify the HTTPS endpoint and who operates it.
- Use OAuth or another approved short-lived credential mechanism.
- Confirm issuer, audience, scopes, and token expiry are validated.
- Separate authentication from per-tool and per-resource authorisation.
- Test tenant isolation with deliberately cross-tenant identifiers.
- Set timeouts, request limits, and rate limits.
- Make mutating operations idempotent where retries are possible.
- Redact secrets and personal data from logs and traces.
- Treat server-supplied text as untrusted content.
- Define how access is revoked and how incidents are audited.

## Links

- [Remote example](https://github.com/garrardkitchen/mcp-server-example)
- [Remote server walk-through](/blog/mcp-server-example/)
- [MCP authorisation](https://modelcontextprotocol.io/specification/2025-06-18/basic/authorization)

## Closing thought

Connectivity answers whether a host can call a remote MCP server; identity and authorisation answer whether it should.
