---
title: "Github Action Workflow Starter"
date: 2022-01-09T15:36:45Z
tags: [github action workflow, starter, cruft, azure devops, bamboo, github plans]
draft: true
---

REWRITE!


In this post I discuss:
- How you can reduce complexity
- How to reduce effort (and friction) and minimize cognitive overload when implementing a new process

As part of my mission to migrate our entire on-premise real-estate to Azure I have redefined some of our CICD pipelines.  The goal is to minimize effort and cognitive overload.  One of my tenets is to always simplify and when possible to revisit first principles. As a step in making this happen, we are using GitHub Actions.  

# Prior experience

Before working at Carfinance 247 I had used BitBucket and their Pipelines feature to build and deploy cloud-native solutions.  I favour this approach.  It keeps everything associated with a solution together.  A huge benefit from this approach is that you (or all developers) don't need to learn or maintain yet another system.  I was lucky enough to be invited to GitHub Actions beta program and since have been an advocate of GitHub Actions.  TBH, I like a lot of what GitHub does.  

## A disappointing anecdote

Someone, not too long ago, said to me, "There you go, you got your way in the end. You can keep your GitHub Actions".  Not exactly verbatim but close enough.  This stuck with me due to it's vindictive undertones.  I'm proud to share with, the reader, that never have I once pursued _anything_ during employment, for my own personal gain.  My focus (and concluding recommendations) have always been, and will always be, to benefit the organisation (or development team) and never to myself.  Maybe I live by a different set of values or just plain unawares of dubious practices?  There have been plenty of technical interests that haven't aligned themselves naturally with current employment and so I've pursued those (scratched those itches) in my own time - doesn't everybody?  Afterall, isn't this what _side projects_ are for?  I'm sure GitHub will be littered with side projects!

# Tooling

At Carfinance 247 we use a variety of CICD tooling.  This list includes Bamboo, Azure DevOps as well as GitHub Actions.  We're moving away from **Bamboo** as it's been determined as not fit for purpose; it fails from time to time as well as being painfully slow (to execute, render).  Zero TLC provided.  This however may be due, in part, to it being 2 major versions behind the latest release and being last updated in 2017!  Then there's **Azure DevOps**.  This requires an unwelcome amount of cruft.  For example, creating Service Connections then sharing with projects.  If you're using templates which requires convention over configuration, then you have to rename these service connections instances. Then there's the permissions faff and lack of features like not being able to create global variables?!  which leaves **GitHub Actions** 🥳.  Sadly, we're not on the Enterprise plan and missing out on features that would undoubtedly contribute to further reductions in effort.  This post is orientated to a non-enterprise plan perspective <sup>1</sup>

{{< hint warning >}}
We have also learned that **Azure DevOps** isn't on their [Microsoft] long term roadmap
{{< /hint>}}

# Creating a GHA Workflow starter

Github has excellent online documentation and here's a link that will help you quickly create your GitHub Workflow starters ➡ [create a starter](https://docs.github.com/en/actions/learn-github-actions/creating-starter-workflows-for-your-organization)

This screenshot is demonstrative of the simplicity of what little you need to do to have a starter to share across your organisation:

![](../img/2022-01-09-15-48-03.png).

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

When you then wish to create a new Action workflow, a starter similar to this will appear:

![](../img/2022-01-09-15-47-17.png)

# Benefits of starters

IMO having starters negate the need to have _How to guides_ <sup>2</sup> on how to create GH Actions from scratch.  This might not be problematic from a green-field perspective but if you're migrating 130+ applications then the last thing you want to be doing (or asking others to do is) cut & pasting and reading the documentation on how to do this!  Less friction, greater acceptance and reduced complexity.

---

<sup>1</sup> - The Enterprise plan allows you to create private `.github` repository.  With using a private repository, you'll be safe to reference secrets.  If you're not using the Enterprise plan, I'd recommend you to use documentation associated with your starter action and reference arbitrary secrets (eg `${{ secret.<acr-username> }}`) for the sole purpose of searching and replacing with the _actual_ secret.  This approach gives nothing to an `external bad actor`.

<sup>2</sup> - My preference is to use READMEs and other markdown documents in the same repository so everything is as close to the code as possible.  I believe this to be the ultimate repository onboarding DX.
