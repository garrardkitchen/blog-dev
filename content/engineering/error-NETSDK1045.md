---
aliases: [/blog/error-NETSDK1045/]
title: "Diagnosing NETSDK1045"
date: 2022-09-28T20:22:13+01:00
tags: [engineering, dotnet, sdk, visual-studio, build-diagnostics]
---

In this article, you'll learn how to diagnose NETSDK1045 by distinguishing an installed SDK from the SDK selected by the build. That matters because installing another SDK only helps when the failing process can discover and select it.

The error means the active .NET SDK does not support the project's target framework. It does **not** necessarily mean that the required SDK is absent from the machine.

## Start with evidence

Run these commands from the project directory and from the same shell or runner that produces the error:

~~~powershell
dotnet --info
dotnet --list-sdks
dotnet --version
~~~

The list command shows what is installed. The version command shows what the CLI selects for the current directory. Those answers can differ when a global.json pins an older feature band or when SDK lookup is affected by architecture and environment variables.

Then check:

1. The project's TargetFramework or TargetFrameworks value.
2. Every global.json found by walking from the working directory towards the filesystem root.
3. Whether the process is x86, x64, or Arm64 and whether the matching SDK is installed.
4. PATH, MSBuildSDKPath, and DOTNET_ROOT overrides.
5. The SDK supported by the installed Visual Studio version.

## Visual Studio is a special case

Visual Studio supports specific .NET SDK feature bands. A newer standalone SDK can therefore work with dotnet build while an older Visual Studio still fails to load or build the project. For the original .NET 6 case, moving from Visual Studio 2019 to a supported Visual Studio 2022 release resolved the mismatch. That is historical context, not a universal prescription to “upgrade Visual Studio” for every NETSDK1045.

The reliable fix is to align three things: a supported target framework, a compatible SDK, and tooling that can use that SDK. In CI, pin the intended SDK deliberately and print dotnet --info in diagnostic output; on developer machines, keep global.json intentional rather than allowing it to become an invisible constraint.

## References
- [Microsoft: NETSDK1045 diagnostics](https://learn.microsoft.com/dotnet/core/tools/sdk-errors/netsdk1045)
- [.NET SDK selection and global.json](https://learn.microsoft.com/dotnet/core/versions/selection)
- [.NET support policy](https://dotnet.microsoft.com/platform/support/policy)

## Closing thought

NETSDK1045 stops being mysterious when SDK discovery and selection are treated as observable build inputs rather than as properties of the machine in general.
