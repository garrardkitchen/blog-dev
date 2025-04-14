---
title: "Compare Resources From Terraform Plan"
date: 2024-02-04T19:39:24Z
tags: [terraform, powershell, azure]
---

Here's a [link](https://github.com/garrardkitchen/terraform-examples/blob/compare/2024-feb-compare-plan-with-resources/README.md) to a repo I created today, that will be used to host examples of all challenges I encounter that relate to Terraform.  

In this particular sample, I need to list out all those resources that will be created using Terraform where we do not have a state file.  The product of this will be added to sheets, by Resource Group, in an Excel spreadsheet.  Each sheet is an Azure Resource Group.  The rationale for this is that at some point an apply was executed, and due to the state not being managed, we do not know if the current config matches with what has been deployed.  

The Excel spreadsheet gives us something organised and visual to use to compare.  This will also be used to confirm the correct naming convensions have been used.


This is an example of the resulting Excel spreadsheet:

![](../img/2024-02-05-06-58-01.png)

To see the the few LOC used to generate this, please [click here](https://github.com/garrardkitchen/terraform-examples/blob/compare/2024-feb-compare-plan-with-resources/README.md).