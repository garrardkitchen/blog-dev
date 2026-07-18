---
aliases: [/blog/inner-loop-qodana-vscode/]
title: "Inner loop with Code Quality with Qodana"
date: 2024-07-07T19:24:20+01:00
tags: [engineering, vscode, ci, "continuous integration", "clean code", "code quality", qodana, jetbrains, Inner-loop, ".net aspire"]
draft: true
---


In this article, you'll learn how to align editor analysis, Qodana configuration, and CI so quality findings arrive consistently. That matters because durable engineering comes from understanding trade-offs, not merely reproducing a command or pattern.

As part of the Inner-loop of a sample .NET Aspire demo solution...

![alt text](/blog/img/qodana-sample-1.png)

...after:

![alt text](/blog/img/qodana-sample-2.png)

# Create connection in vscode

need to deploy via GH first to obtain project id in url path. VSCode requires the project id for connection

The `qodana.yml` file must live in the root.

```yaml
exclude:
  - name: All
    paths:
      - Samples/Sample_Aspire_Container_Redis/Sample_Aspire_Container_Redis.ServiceDefaults/
      - Samples/Sample_Aspire_Container_Redis/Sample_Aspire_Container_Redis.Web/Components/App.razor
      - Samples/Sample_Aspire_Container_Redis/Sample_Aspire_Container_Redis.Web/wwwroot/
      - Samples/Sample_Aspire_Container_Redis/Sample_Aspire_Container_Redis.Web/Components/Layout/MainLayout.razor.css
  - name: ArrangeObjectCreationWhenTypeNotEvident
  - name: ClassNeverInstantiated.Global
  - name: NotAccessedPositionalProperty.Global
  - name: UnusedMember.Global
```

# linter that works

_jetbrains/qodana-cdnet:2024.1-eap didn't_

```
linter: jetbrains/qodana-dotnet:2024.1
```

# set solution file

## Errors


- Run Locally:

  ![alt text](/blog/img/qodana-error-runlocally.png)


- Run via the cli:

   ![alt text](/blog/img/qodana-error-runcli.png)


I didn't find the answer in their documentation. I found the answer on YT. I later found [this](https://www.jetbrains.com/help/qodana/qodana-dotnet-docker-readme.html#Analyzing+specific+solution+or+project). I wasted a few hours in this as I wanted to run this locally as it was important to me to be able to run this as part of my inner-loop. I would add this to my [taskfile](https://taskfile.dev/).

```
dotnet:
  solution: Samples/Sample_Aspire_Container_Redis/Sample_Aspire_Container_Redis.sln
```

# GitHub workflow - how to set the project if using a monorepo

`with args`


```yml
name: Qodana
on:
  workflow_dispatch:
  pull_request:
  push:
    branches: # Specify your branches here
      - main # The 'main' branch
      - kitcheng/*

jobs:
  qodana:
    runs-on: ubuntu-latest
    permissions:
      contents: write
      pull-requests: write
      checks: write
    steps:
      - uses: actions/checkout@v3
        with:
          ref: ${{ github.event.pull_request.head.sha }}  # to check out the actual pull request commit, not the merge commit
          fetch-depth: 0  # a full history is required for pull request analysis
      - name: 'Qodana Scan'
        uses: JetBrains/qodana-action@v2024.1
        with:
          args: --project-dir,Samples/Sample_Aspire_Container_Redis/
        env:
          QODANA_TOKEN: ${{ secrets.QODANA_TOKEN }}
```

# install the qodana cli

_downloads the docker image_

```powershell
C:\Users\garra\AppData\Roaming\Code\User\globalStorage\jetbrains.qodana-code\ncc9l\qodana.exe scan -e QODANA_TOKEN="<project-token>" --project-dir=Samples/Sample_Aspire_Container_Redis/
```

# Enable Roslyn Analyzers

...

# Observations

- Plenty of visual noise so ended up adding several inspections to the exclude list
- Patterns feature eg *.css didn't work for me, in fact the report gave a bill of healthy when I knew that was not the case


## Deepening the article

## Make local and CI analysis comparable

Start with one Qodana configuration at the repository boundary and run the same linter family locally and in CI. In a monorepo, set the project directory explicitly and make the path relative to the checkout, not to a developer's machine. Pin the Qodana image or action version so a ruleset change does not arrive as an unexplained build failure.

IDE inspection, Roslyn analyzers, and Qodana overlap without being identical. Decide which findings block a merge, which are advisory, and where suppressions live. A suppression should capture why the finding is acceptable and, where appropriate, when that decision expires.

Quality gates should compare against a baseline when introducing analysis to an established codebase. Otherwise thousands of inherited findings obscure the regression introduced by the current change. Ratchet the baseline down deliberately, publish the report as a build artifact, and keep credentials for Qodana Cloud in repository or environment secrets rather than in the YAML file.

## References

- [YAML file](https://www.jetbrains.com/help/qodana/qodana-yaml.html#Include+an+inspection+into+the+analysis+scope)
- [Qodana documentation](https://www.jetbrains.com/help/qodana/)
- [Qodana .NET documentation](https://www.jetbrains.com/help/qodana/dotnet.html)

## Closing thought

Qodana strengthens the inner loop only when an editor finding and a CI finding tell the same story, at a time when the author can still act on it.
