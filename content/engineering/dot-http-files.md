---
aliases: [/blog/dot-http-files/]
title: "A practical guide to .http files"
date: 2026-07-08T07:49:14+01:00
draft: false
featured: true
description: A Practical Guide to .http Files - Ditch Postman, Keep Your API Calls in Source Control
tags: [engineering, .http, good engineering, api, vscode, rider]
---

_A Practical Guide to .http Files - Ditch Postman, Keep Your API Calls in Source Control_

If you've spent any time bouncing between a browser tab, Postman, and your terminal just to test an endpoint, I've got good news: there's a simpler way, and it lives right next to your code. `.http` files let you write, save, and share HTTP requests as plain text — version-controlled, reviewable in a pull request, and runnable with a single click in VS Code, Visual Studio, JetBrains IDEs, or via CLI tools like `httpyac`.

I use these on every project now, from quick API prototyping to full integration test suites. Here's how to get the most out of them.

## What Is a .http File?

It's exactly what it sounds like: a text file containing one or more HTTP requests, written in a readable, curl-like syntax. No JSON config, no proprietary export format — just a file you can open, edit, and diff like any other source file.

```
GET https://api.github.com/users/octocat
Accept: application/json
```

Save that as `requests.http`, click "Send Request" (in VS Code with the REST Client extension, or natively in Visual Studio and Rider), and you get a response pane right there in your editor.

> [!TIP]
> Visual Studio and Rider support `.http` files natively — no extension required. In VS Code, install the **REST Client** extension by Huachao Mao.

## Why Bother? The Strengths Are Real

**They live in your repo.** Your API contract examples travel with the code that implements them. New team members can read `auth.http` and immediately understand how login, token refresh, and logout work — no onboarding wiki required.

**They're diffable and reviewable.** When an endpoint changes, the `.http` file change shows up in the pull request. That's a huge win for API discoverability compared to a request buried in someone's personal Postman workspace.

**No account, no sync, no vendor lock-in.** Nothing to install beyond your editor (or a lightweight extension), nothing syncing to a third-party cloud.

> [!NOTE]
> The biggest win here isn't the syntax — it's the workflow shift. API examples become part of your codebase's history instead of living in someone's personal tooling, invisible to the rest of the team.

## Core Syntax

Multiple requests in one file, separated by `###`:

```
### Get a user
GET https://api.github.com/users/octocat
Accept: application/json

### Create a widget
POST https://api.example.com/widgets
Content-Type: application/json

{
  "name": "Gizmo",
  "quantity": 10
}
```

Each `###` line starts a new named block. Add a comment above it for clarity — most tools display that as the request's display name in a request list.

## Variables and Environments

This is where `.http` files go from "handy" to "genuinely production-ready." Define variables at the top of the file:

```
@baseUrl = https://api.example.com
@token = {{$dotenv AUTH_TOKEN}}

### Get widgets
GET {{baseUrl}}/widgets
Authorization: Bearer {{token}}
```

For multiple environments (dev, staging, prod), how you configure this depends on your editor — this trips people up because the two major tools don't share a mechanism.

**VS Code (REST Client extension):** environments are defined in your `settings.json`, under the `rest-client.environmentVariables` key:

```json
{
  "rest-client.environmentVariables": {
    "dev": {
      "baseUrl": "https://dev-api.example.com"
    },
    "prod": {
      "baseUrl": "https://api.example.com"
    }
  }
}
```

You then pick the active environment from the status bar or via the command palette (`Rest Client: Switch Environment`).

**Rider / IntelliJ-based IDEs:** environments live in one or two JSON files sitting alongside your `.http` files:

- `http-client.env.json` — the **public** file, safe to commit, for non-sensitive values shared across the team (base URLs, non-secret config).
- `http-client.private.env.json` — the **private** file, for anything sensitive (tokens, passwords). This one should be `.gitignore`d.

```json
// http-client.env.json (commit this)
{
  "dev": {
    "baseUrl": "https://dev-api.example.com"
  }
}
```

```json
// http-client.private.env.json (do NOT commit this)
{
  "dev": {
    "authToken": "your-local-dev-token"
  }
}
```

Rider automatically merges both files for the selected environment, and you switch environments from a dropdown in the HTTP client gutter.

> [!NOTE]
> Neither `settings.json` nor `http-client.env.json`/`http-client.private.env.json` are interchangeable between VS Code and Rider — they're editor-specific mechanisms. If your team uses a mix of editors, you'll need to maintain both, or standardise on one tool for shared `.http` workflows.

Whichever mechanism you use, the payoff is the same: switch environments in one place, and every request in the file re-targets automatically. That's a much safer pattern than hardcoding URLs — you eliminate the classic "oops, I just hit prod" mistake.

## Chaining Requests

You can capture a response and reuse it in a later request — genuinely useful for auth flows:

```
### Login
# @name login
POST {{baseUrl}}/auth/login
Content-Type: application/json

{
  "username": "demo",
  "password": "{{$dotenv DEMO_PASSWORD}}"
}

### Use the token from login
GET {{baseUrl}}/profile
Authorization: Bearer {{login.response.body.$.token}}
```

The `# @name` comment labels a request so you can reference its response body, headers, or status elsewhere in the file.

## A Word on Security

A few habits worth building in from day one:

- **Never commit secrets.** In VS Code, keep tokens out of `settings.json` if that file is shared/synced, and prefer `{{$dotenv VAR_NAME}}` with a `.gitignore`d `.env` file instead. In Rider, put secrets only in `http-client.private.env.json`, never in the public `http-client.env.json`.
- **Keep prod credentials out of shared, committed files.** The public `http-client.env.json` (Rider) or a team-shared `settings.json` snippet (VS Code) should never carry real production tokens — only non-sensitive config like base URLs.
- **Treat these files like any other test artifact in code review.** If a request block hits a destructive endpoint (`DELETE`, bulk operations), flag it the same way you'd flag any other risky test code.

> [!WARNING]
> A leaked token in a `.http` file is just as exposed as one hardcoded in application code. Git history doesn't forget — if a secret does slip into a commit, rotate it immediately rather than just removing it in a follow-up commit.

> [!IMPORTANT]
> Add `.env` (VS Code) and `http-client.private.env.json` (Rider) to `.gitignore` *before* your first commit in a new repo, not after. It's a two-second habit that prevents an incident-response afternoon.

## Where They Fit Best

- **API exploration** during development — faster feedback loop than Swagger UI for anything beyond trivial requests.
- **Living documentation** for your team's internal APIs, sitting in the same repo as the service.
- **Smoke-testing deployed environments** after a release, using the environment-switching feature to hit dev, staging, and prod in turn.

> [!CAUTION]
> Don't reach for `.http` files as a substitute for real integration or contract tests. They're excellent for exploration and documentation, but they won't catch regressions on their own — keep your automated test suite as the source of truth for correctness.

They're not a full replacement for a proper automated test suite or a tool like Postman/Insomnia when you need elaborate scripting, mocking, or team collaboration features at scale. But for the day-to-day of "let me just check this endpoint," they've earned a permanent place in my toolkit — and I'd encourage you to give them a real trial on your next project.