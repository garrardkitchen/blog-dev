---
title: "PowerShell Tips and Tricks"
date: 2022-02-11T05:58:03Z
draft: true
tags: [engineering, powershell, tips, ValidateSet]
---


In this article, you'll learn how a few PowerShell idioms improve discoverability, validation, and pipeline readability. That matters because durable engineering comes from understanding trade-offs, not merely reproducing a command or pattern.

## Naming convention

## Select-Object

```powershell
Get-Service | Select-Object Name
```

## Where-Object

```powershell
Get-Service | Where-Object Status -eq Stopped
```

##

## ValidateSet

If you want to remove any possibility of an input being incorrectly provided, you can set the range of values that are acceptable from a parameter by using the `ValidateSet Param` attribute.

```powershell
Function Test-ValidateSet() {
    Param(
        [Parameter(Mandatory=$true)]
        [ValidateSet("Garrard", "Louise", "Charles", "Edwards")]
        [string]$name
    )

    if ($name -eq "Garrard") {
        Write-Host "Correct, you entered $name" -ForegroundColor Green
    } else {
        Write-Host "Incorrect, you entered $name" -ForegroundColor Red
    }
}
```

Loops

Conditions

JSON

XML

Casting

Exception handling

## Deepening the article

## Design for the pipeline

PowerShell passes objects through the pipeline, so postpone formatting until the edge. Select-Object creates projected objects; Format-Table produces formatting instructions intended for display. Returning formatted output from a reusable function makes later filtering and export unnecessarily difficult.

Use approved Verb-Noun names for functions, singular nouns, and parameter types that express intent. ValidateSet is useful for a small stable vocabulary:

~~~powershell
function Get-DeploymentStatus {
  [CmdletBinding()]
  param(
    [Parameter(Mandatory)]
    [ValidateSet('Development', 'Test', 'Production')]
    [string] $Environment
  )

  Get-Content "./status/$Environment.json" -Raw |
    ConvertFrom-Json
}
~~~

For values that change independently, ValidateScript, argument completers, or dynamic discovery may be a better contract. Add SupportsShouldProcess to commands that change state so -WhatIf and -Confirm work predictably. Prefer terminating errors for failures a caller must handle, and include enough context to diagnose the operation without printing secrets.

## Closing thought

PowerShell is at its clearest when functions return useful objects, parameters express their contract, and formatting is left to the person at the end of the pipeline.
