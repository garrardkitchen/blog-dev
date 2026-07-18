---
aliases: [/blog/deploy-to-azure-button/]
title: "Deploy to Azure Button"
date: 2022-03-18T11:09:29Z

tags: [cloud, azure, arm-templates, developer-experience]
draft: true
---


In this article, you'll learn how a Deploy to Azure badge turns a repository template into a repeatable portal deployment experience. That distinction matters because cloud failures usually emerge at the seams between configuration, identity, networking, and operations.

Have you ever thought about adding a Deploy to Azure button to your ARM deployment README file?

# Step 1

Navigate to your ARM azuredeploy.json file and press the `Raw` button.

![](/blog/img/2022-03-18-11-12-26.png)

Your browser's address bar will contain a URL similar to `https://raw.githubusercontent.com/<path-to-deploy>/azuredeploy.json`

Copy this [URL] into your clipboard

# Step 2

Escape the URL by using the `EscapeDataString` function:

```powershell
$url = "https://raw.githubusercontent.com/<path-to-deploy>/azuredeploy.json"
[uri]::EscapeDataString($url)
```

# Step 3

Add to markdown

```powershell
[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2F%3Cpath-to-deploy-escaped%3E%2Fazuredeploy.json)
```

When added to your markdown file, the above markdown will produce this:

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2F%3Cpath-to-deploy-escaped%3E%2Fazuredeploy.json)

## Deepening the article

## Make the badge reproducible

The portal link accepts the URI of a publicly reachable ARM template. Put the template at an immutable release URL when possible; linking to a moving branch means the same badge can deploy different infrastructure tomorrow.

~~~markdown
[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](
  https://portal.azure.com/#create/Microsoft.Template/uri/ENCODED_TEMPLATE_URI
)
~~~

Encode only the template URI, not the entire portal URL. Test the result in a private browser session so cached Azure Portal state does not hide a malformed link.

A public template cannot contain secrets, and parameters marked secureString or secureObject protect values from ordinary deployment logging rather than making unsafe defaults acceptable. Prefer managed identity and Key Vault references. Also provide a command-line deployment route for repeatability; a portal button is an onboarding convenience, not a deployment pipeline.

For templates that need linked artifacts, use stable, accessible URIs or package the deployment using Bicep, Template Specs, or a deployment stack suited to the operating model.

## References
- [Microsoft Azure documentation](https://learn.microsoft.com/azure/)
- [Microsoft identity platform documentation](https://learn.microsoft.com/entra/identity-platform/)

## Closing thought

A Deploy to Azure button removes friction from the first deployment; reproducible templates and explicit identity are what make the second deployment trustworthy.
