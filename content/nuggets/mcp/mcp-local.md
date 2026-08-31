---
date: 2025-04-17
title: "Local MCP Checklist"
description: "A compact checklist for reviewing a local MCP server before enabling it."
tags: [ai, model-context-protocol, local-development, checklist]
---

This checklist helps you review a local MCP server's command, permissions, transport, and failure behavior before enabling it. That matters because the process can often act with the same filesystem and credential access as its host.

## Checklist

- Read the configured command, arguments, and working directory.
- Pin or verify the package or executable you are launching.
- Keep JSON-RPC messages on standard output; route diagnostics to standard error.
- Pass only the environment variables the server needs.
- Prefer read-only tools and narrow path allow-lists.
- Validate tool arguments and cap output size.
- Require confirmation for writes, commands, or external side effects.
- Confirm how the host stops orphaned or unresponsive processes.
- Inspect logs for secrets before sharing diagnostics.

## Links

- [Local example](https://github.com/garrardkitchen/mcp-example)
- [MCP transports](https://modelcontextprotocol.io/specification/2025-06-18/basic/transports)
- [VS Code MCP servers](https://code.visualstudio.com/docs/copilot/chat/mcp-servers)

## Closing thought

A short local MCP configuration can launch a highly capable process, so its review should be proportional to the capability—not to the number of lines.
