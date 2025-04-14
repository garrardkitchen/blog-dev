---
title: "Powershell Tips and Tricks"
date: 2022-02-11T05:58:03Z
draft: true
tags: [powershell, tips, ValidateSet]
---

## Naming convension

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

