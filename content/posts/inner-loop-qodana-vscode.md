---
title: "Inner loop with Code Quality with Qodana"
date: 2024-07-07T19:24:20+01:00
tags: [vscode, ci, continuous integration, clean code, code quality, qodana, jetbrains, Inner-loop, .net aspire]
draft: true
---

As part of the Inner-loop of a sample .NET Aspire demo solution...

![alt text](../img/qodana-sample-1.png)

...after:  

![alt text](../img/qodana-sample-2.png)

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

  ![alt text](../img/qodana-error-runlocally.png)


- Run via the cli:

   ![alt text](../img/qodana-error-runcli.png)


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


# References

- [YAML file](https://www.jetbrains.com/help/qodana/qodana-yaml.html#Include+an+inspection+into+the+analysis+scope)