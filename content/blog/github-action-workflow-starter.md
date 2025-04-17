---
title: "Github Action Workflow Starter"
date: 2022-01-09T15:36:45Z
tags: [github action workflow, starter, cruft, azure devops, bamboo, github plans]
draft: false
---

In this post, I share practical insights on how to simplify and streamline your CI/CD processes using GitHub Actions. The focus is on reducing complexity, minimizing effort, and making it easier for teams to adopt new workflows with less cognitive overhead.

## Background

During our migration from on-premise infrastructure to Azure, I had the opportunity to revisit and redesign our CI/CD pipelines. My guiding principle throughout this process was to keep things as simple as possible, always returning to first principles when evaluating tooling and process changes. GitHub Actions emerged as a strong candidate for our needs, offering flexibility and ease of use.

## Previous Experience

Before joining Carfinance 247, I worked with BitBucket Pipelines to build and deploy cloud-native solutions. I appreciated how this approach kept everything related to a solution in one place, reducing the need to learn or maintain additional systems. My early access to GitHub Actions further reinforced my preference for integrated, developer-friendly tooling.


## An anecdote

During my time working with different teams, there have been moments of differing opinions about tooling choices. For example, when GitHub Actions was adopted, some colleagues remarked on my enthusiasm for the platform. For me, the motivation has always been to select tools and processes that best serve the needs of the organization and the development team as a whole. Like many in the industry, I also enjoy exploring new technologies and working on side projects in my own time—it's a common way for developers to learn and grow outside of their day-to-day responsibilities.

## Tooling Landscape

At Carfinance 247, we have used a variety of CI/CD tools, including Bamboo, Azure DevOps, and GitHub Actions. Over time, we found that Bamboo was no longer meeting our needs due to reliability and performance issues. Azure DevOps, while powerful, introduced additional complexity with service connections, permissions, and configuration overhead. GitHub Actions, on the other hand, provided a more streamlined experience, even though some advanced features are only available on the Enterprise plan.

{{< callout type="warning" >}}
We have also learned that Azure DevOps is not on Microsoft's long-term roadmap.
{{< /callout >}}

## Creating a GitHub Actions Workflow Starter

GitHub offers excellent documentation for creating workflow starters. You can find a helpful guide here: [create a starter](https://docs.github.com/en/actions/learn-github-actions/creating-starter-workflows-for-your-organization)

This screenshot is demonstrative of the simplicity of what little you need to do to have a starter to share across your organisation:

![](../img/2022-01-09-15-48-03.png)

To create a workflow starter, set up a public repository named `.github` with a `workflow-templates` folder. Place your starter Action YAML files there—each can have an associated `*.properties.json` file to provide metadata and control when the starter is suggested. File patterns in the properties file allow you to offer relevant starters based on the contents of a repository.



Essentially you're creating a public repository named `.github` with a folder named `workflow-templates`.  Contained within is your starter Action .yaml file.  You can have many.  Each Action file needs to have an associated `*.properties.json` file that labels and describes your starter.  What adds to the depth of the relatively simple yet powerful feature is the ability to add intelligence to what starters are offered up to the author when creating an Action.  This is made possible by the use of **filepatterns**.  Anything that matches a file pattern in the `root` will predicate whether that starter is offered up or not.  Clever hey?  I've included a sample `*.properties.json` for context below:

```json
{
    "name": "Deploy Workflow",
    "description": "Deploy to AKS",
    "iconName": "azure-icon",
    "categories": [
        "csharp"
    ],
    "filePatterns": [
        "package.json$",
        "^Dockerfile",
        ".*\\.md$"
    ]
}
```

When creating a new Action workflow, GitHub will suggest appropriate starters based on these patterns, making it easier for teams to get started quickly and consistently:

![](../img/2022-01-09-15-47-17.png)

## Benefits of Workflow Starters

Workflow starters reduce the need for extensive documentation or step-by-step guides, especially when migrating many applications. By providing ready-to-use templates, you lower the barrier to entry, encourage best practices, and help teams focus on delivering value rather than wrestling with configuration.

---

<sup>1</sup> The Enterprise plan allows you to create private `.github` repositories, which is useful for referencing secrets securely. If you're not on the Enterprise plan, consider documenting how to handle secrets in your starter templates, using placeholders that can be replaced as needed.

<sup>2</sup> My preference is to keep documentation close to the code, using READMEs and markdown files within the repository to support onboarding and knowledge sharing.
