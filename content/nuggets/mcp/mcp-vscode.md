---
date: 2025-04-17
title: "VS Code MCP Checklist"
description: "A compact checklist for enabling MCP servers in VS Code."
tags: [ai, model-context-protocol, vscode, checklist]
---

This checklist shows what to inspect before enabling an MCP server in VS Code and how to keep tool use observable. That matters because a server can turn a chat interaction into a real operation on local or remote systems.

## Checklist

- Review the server definition committed to the workspace.
- Do not approve a command merely because the repository is trusted.
- Check executable paths, arguments, environment input, and endpoint ownership.
- Prefer input variables or approved secret storage over literal credentials.
- Inspect the server's discovered tools and disable capabilities you do not need.
- Read tool arguments before approving consequential calls.
- Keep workspace trust, extension trust, and MCP server trust as separate decisions.
- Re-check configuration changes during code review.
- Stop or remove servers that are no longer needed.

## Links

- [VS Code MCP server documentation](https://code.visualstudio.com/docs/copilot/chat/mcp-servers)
- [MCP architecture](https://modelcontextprotocol.io/docs/learn/architecture)

## Closing thought

The most important action in a tool-enabled editor is often the pause before deciding what has earned permission to run.
