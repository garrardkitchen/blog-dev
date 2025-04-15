---
title: "Error NETSDK1045"
date: 2022-09-28T20:22:13+01:00
tags: [error, visual studio, vs, vs2019, .net, sdk]
---

#  Error NETSDK1045 The current .NET SDK does not support targeting .NET 6.0

A colleague had Visual Studio shout this (see 👇) at him when he loaded up a FunctionsApp project.  He received this error twice as the second project was the unit tests for the FunctionsApp.

{{< hint danger>}}

Error  NETSDK1045  The current .NET SDK does not support targeting .NET 6.0.  Either target .NET 5.0 or lower, or use a version of the .NET SDK that supports .NET 6.0.    TestNasLinuxFuncAppWebTests    
C:\Program Files\dotnet\sdk\5.0.409\Sdks\Microsoft.NET.Sdk\targets\Microsoft.NET.TargetFrameworkInference.targets    141    
{{< /hint >}}

I interpretted this as the newer targets were not supported by the existing SDK.  Yeah, genius right 😁.

However, even after installing .NET 6.0 and the Azure Functions Core Tools using it didn't fix the issue:

```
winget install -e --id Microsoft.DotNet.SDK.6
winget install -e --id Microsoft.AzureFunctionsCoreTools
```

So, like most at this stage, I consulted with a trusted colleauge (Google) and found this in the MS Documentation - https://learn.microsoft.com/en-us/dotnet/core/tools/sdk-errors/netsdk1045.  He did reboot his vm, yet the error remained.  

I did also find a SO suggesting upgrading to VS2022 would fix this issue.  

I couldn't find the reciprocal recommendation in MS documentation but he did upgrade to vs2022 regardless.  This did the trick. 🥳

**Conclusion**

If you get **Error NETSDK1045** in Visual Studio then upgrade to the latest verison of VisualStudio.


