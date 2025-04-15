---
title: "Azure Devops Artifacts Upstream Source Oddness"
date: 2022-09-30T10:44:20+01:00
tags: [nuget, azure, devops, artifacts, upstream source]
draft: true
---

Apologies in advance for the bitiness of this post.  It's regarding nuget which sometimes can be a bit of a black art 😁.  It saddens me to admit that I once won a NuGet book but it didn't survive my last book cull...or get read ... awkward 🤭

Ok then, this is what transpired...I recently developed a project outside of a client's Azure Subcription.  When I was ready to deploy the infra, application and pipelines into a client's AzDevOps instance, a colleague first pulled the private git repo to his Azure VM development machine.

When we called `dotnet restore`, it failed to restore the applications (and test projects) Nuget package dependencies from the upstream source.  😕 We confirmed, the Azure DevOps Aftifact Nuget Feed did have the Nuget Gallery as an Upstream Source.  

no nuget.config in these projects but there was one ... correct the rest of this!!!

We checked the `nuget.config` file and this only referenced the Azure DevOps Artifact feed.  

We also checked:
- local packages in `C:\Users\<username>\.nuget\packages` to see if one of the dependencies could have been pulled in from here - there wasn't
- nuget.config in `C:\Users\<username>\AppData\Roaming\NuGet` to see if there was a refernce to the nuget gallery (https://api.nuget.org/v3/index.json) - there wasn't.

To get beyond this, we added the Nuget uri as a new *packageSources* in the nuget.config file. 

Perplex and mildlt embarrassed, I took this offline and investigated.  I had a personal azdevops org instance so I created a new Artifact Nuget feed with the same Nuget Upstream Source.

## Some observsations

Here are my observations:

```text
- You CANNOT add package from private feed
```

{{< hint warning example>}}
dotnet add package xunit --version 2.4.2
{{</ hint>}}

👆 This example does not add a cached version of this package to your private feed

👆 You can only add your own nuget packages to your private feed

```
- You CAN restore a nuget from a Upstream Source. 
```

👆 Doing a restore will add the Upstream Nuget package it to your private feed.

👆 The package ref must first exist in the csproj file.

## A sequence

I put together a sequence to capture what was happening.  I'm hoping will be clear enough to understand.  Spoiler alert, there is a level of oddness about it:

	- Sequence:
		○ Add from api.nuget.org - `dotnet add package Moq --version 4.18.2 -s https://api.nuget.org/v3/index.json`
		○ Remove .nuget\packages\moq
		○ Then `dotnet restore`:
			§ You get :
			 dotnet restore                                                                                     in pwsh at 10:28:23
			  Determining projects to restore...
			  Retrying 'FindPackagesByIdAsync' for source 'https://pkgs.dev.azure.com/garrardkitchen/_packaging/e859c2f8-bcb9-4e6e-bfe1-6fc8e54e7ac3/nuget/v3/f
			  lat2/moq/index.json'.
			  Response status code does not indicate success: 401 (Unauthorized).
			  Retrying 'FindPackagesByIdAsync' for source 'https://pkgs.dev.azure.com/garrardkitchen/_packaging/e859c2f8-bcb9-4e6e-bfe1-6fc8e54e7ac3/nuget/v3/f
			  lat2/moq/index.json'.
			  Response status code does not indicate success: 401 (Unauthorized).
			  Retrying 'FindPackagesByIdAsync' for source 'https://pkgs.dev.azure.com/garrardkitchen/_packaging/e859c2f8-bcb9-4e6e-bfe1-6fc8e54e7ac3/nuget/v3/f
			  lat2/moq/index.json'.
			  Response status code does not indicate success: 401 (Unauthorized).
			  Retrying 'FindPackagesByIdAsync' for source 'https://pkgs.dev.azure.com/garrardkitchen/_packaging/e859c2f8-bcb9-4e6e-bfe1-6fc8e54e7ac3/nuget/v3/f
			  lat2/moq/index.json'.
			  Response status code does not indicate success: 401 (Unauthorized).
			  Retrying 'FindPackagesByIdAsync' for source 'https://pkgs.dev.azure.com/garrardkitchen/_packaging/e859c2f8-bcb9-4e6e-bfe1-6fc8e54e7ac3/nuget/v3/f
			  lat2/moq/index.json'.
			  Response status code does not indicate success: 401 (Unauthorized).
			C:\Users\KITCHENG\source\pg\nuget\nuget.csproj : error NU1301: Failed to retrieve information about 'Moq' from remote source 'https://pkgs.dev.azur
			e.com/garrardkitchen/_packaging/e859c2f8-bcb9-4e6e-bfe1-6fc8e54e7ac3/nuget/v3/flat2/moq/index.json'.
			  Failed to restore C:\Users\KITCHENG\source\pg\nuget\nuget.csproj (in 7.25 sec).
		○ Then comment out package ref in csproj
		○ Then `dotnet restore`:
			 dotnet restore                                                                                     in pwsh at 10:28:33
			  Determining projects to restore...
			  All projects are up-to-date for restore.
		○ Then uncomment and `dotnet restore` again
			dotnet restore                                                                                     in pwsh at 10:28:36
			  Determining projects to restore...
			  Restored C:\Users\KITCHENG\source\pg\nuget\nuget.csproj (in 3.18 sec).
		○ Then finally, package appears in devops nuget feed


---

