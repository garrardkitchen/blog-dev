---
linkTitle: MCP
title: Model Context Protocol 
description: Articles explaining MCP concepts, local and remote servers, and secure AI integrations.
# prev: /docs/guide/shortcodes/tabs
# next: /docs/advanced/multi-language
draft: false
layout: category-list
---

## What is Model Context Protocol (MCP)

Model Context Protocol (MCP) is an open protocol that standardizes how applications supply context to large language models (LLMs), similar to how USB-C standardizes device connections. It enables AI tools to integrate with local and remote data sources, supporting agent workflows, flexibility between vendors, and secure handling of user data. MCP follows a client-server architecture with hosts, clients, and servers working together.

## What Problem Does MCP Solve?

Traditionally, AI assistants and code tools operate in silos, each with their own way of understanding project context (such as open files, project structure, or user intent). This fragmentation leads to:
- Inconsistent or incomplete context for AI models
- Redundant or conflicting integrations
- Security and privacy concerns around context sharing

MCP provides a standardized way for tools to communicate context, improving interoperability, privacy, and the quality of AI-powered suggestions.  It is based on the **client-server** architecture, where a host application can connect to multiple servers

## Evolution and Adoption

MCP is still in active development, with early adoption by some editor extensions and AI platforms. The protocol is being discussed and refined by the open-source community, with growing interest from major AI tool vendors and IDE developers. As the protocol matures, broader adoption is expected, leading to more seamless and secure AI integrations across tools.

For the latest updates, specifications, and community discussions, visit the official [Model Context Protocol (MCP) website](https://modelcontextprotocol.io/introduction).

---

**References:**
- [Model Context Protocol (MCP) Home](https://modelcontextprotocol.io/introduction)
- [MCP GitHub Organization](https://github.com/modelcontextprotocol)

<!--more-->

{{< cards >}}
  {{< card link="mcp-local" title="Local" icon="translate" >}}
  <!-- {{< card link="mcp-remote" title="Remote" icon="pencil" >}}
  {{< card link="mcp-vscode" title="VSCode System" icon="chat-alt" >}} -->
{{< /cards >}}
