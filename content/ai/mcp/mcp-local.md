---
date: 2025-04-17
title: "Building a Local MCP Server"
description: "A practical entry point for running an MCP server as a local process."
tags: [ai, model-context-protocol, local-development, security]
---

In this article, you'll learn where the local MCP example lives, how standard input/output transport works, and what trust decisions remain with the user. That matters because a local server inherits the permissions of the process that launches it.

A local MCP server is commonly started by the host as a child process. Protocol messages travel over standard input and standard output. This is convenient—there is no listening port or separate service deployment—but it is not a sandbox.

## Example and operating model

The [local MCP example](https://github.com/garrardkitchen/mcp-example) explores a small server and its host configuration. When adapting it:

- keep protocol messages on standard output and send diagnostics to standard error;
- use absolute executable and working-directory paths where host configuration may vary;
- pin dependencies and review the command before enabling it;
- request the narrowest filesystem, network, and credential access possible; and
- validate every tool argument even when the caller is local.

Environment variables passed by the host become available to the child process. Do not pass broad cloud credentials merely because the process is on your workstation. Prefer a short-lived or workload-scoped identity and keep consequential tool calls behind user confirmation.

## When local is the right choice

Local transport is well suited to development tools, access to a checked-out repository, and experiments that should not expose a network service. A remote server is more appropriate when capabilities must be shared, centrally maintained, or separated from the user's machine. That move introduces authentication, authorisation, tenant isolation, and network failure; it is an architectural change, not only a transport setting.

## References
- [MCP architecture](https://modelcontextprotocol.io/docs/learn/architecture)
- [MCP transports](https://modelcontextprotocol.io/specification/2025-06-18/basic/transports)
- [VS Code MCP servers](https://code.visualstudio.com/docs/copilot/chat/mcp-servers)

## Closing thought

A local MCP server may be one process away, but every tool it exposes is still a decision about how much of the user's machine an AI interaction should be allowed to reach.
