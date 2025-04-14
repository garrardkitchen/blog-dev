---
title: "Deploy to Azure Button"
date: 2022-03-18T11:09:29Z
draft: true
---

Have you ever thought about adding a Deploy to Azure button to your ARM deployment README file?  

# Step 1 

Navigate to your ARM azuredeploy.json file and press the `Raw` button.

![](img/2022-03-18-11-12-26.png)

Your browser's address bar will contain a URL simiar to `https://raw.githubusercontent.com/<path-to-deploy>/azuredeploy.json`

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
