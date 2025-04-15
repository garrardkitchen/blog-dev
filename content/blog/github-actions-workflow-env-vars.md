---
title: "Github Actions Workflow Env Vars"
date: 2022-01-08T15:13:52Z
tags: [github actions, linux, windows, syntax, self-hosted runner, environment variables, workflow, cicd]
---

In my current role as Head of Cloud Platform, I am leading the technical effort of migrating our entire on-premise real-estate to Azure.  Part of this mission, is to upgrade the runtimes of our applications, regardless of their current placement; IIS Web apps, Windows Services and Docker Swarm containers.  I say "part of this mission" as another aspect of this migration is to create a new foundation for our platform - AKS.  I hope to cover more on this in later posts.

# Github workflows

We are using Self-Hosted Runners to build and deploy our applications to AKS.  We have a Hub&Spoke network architecture and our AKS clusters are private.  We have other backing services that are deliberately behind Azure Private Endpoints.  Our architecture enables us to deploy securely from our company network to our spoke VNETs that exist across our Azure Subscriptions.

We're targeting 2 guest operating systems with our containerization orchestration solution - AKS.  These are Linux (.NET Core workloads) and Windows (.NET Framework workloads).  We are having to upgrade our .NET Framework runtimes to 4.8 as this is the minimum requirement to running containers in Kubernetes in Azure.

There are subtle GitHub Actions Workflows expression differences when working with Powershell and bash.  Here in this post I concentrate on how you create and set env vars.

Is the snippet below, which is from our `deploy` GHA workflow, I use a workflow_dispatch to deploy either a feature branch or our main branch.  Feature branches are deployed to our Development Cluster and non-feature branches to our Production Cluster.

```yml
- name: SETUP MAIN BRANCH
  if: ${{ github.ref == 'refs/heads/main' || github.ref == 'refs/heads/master' }}
  run: |    
    echo "ENV=prod" >> $GITHUB_ENV        
    echo "TAG=1.0.${{github.run_number}}" >> $GITHUB_ENV       
    echo "PWD=$(pwd)" >> $GITHUB_ENV        

- name: SETUP FEATURE BRANCH
  if: ${{ github.ref != 'refs/heads/main' && github.ref != 'refs/heads/master' }}
  run: |
    echo "ENV=dev" >> $GITHUB_ENV        
    echo "TAG=${{ github.ref_name }}" >> $GITHUB_ENV
    echo "PWD=$(pwd)" >> $GITHUB_ENV        
```

In the example above, we generate env vars that are used later in this workflow.  The above is running on one of our Linux Self-Hosted Runners so using bash script.

The above example will not update env vars when run on a Windows Self-Hosted Runner (PowerShell).

The equivalent when targeting windows is:

```yml
- name: SETUP MAIN BRANCH
  if: ${{ github.ref == 'refs/heads/main' || github.ref == 'refs/heads/master' }}
  run: |        
    echo "ENV=prod" >> $env:GITHUB_ENV        
    echo "TAG=1.0.${{github.run_number}}" >> $env:GITHUB_ENV       
    echo "PWD=$(Get-Location)" >> $env:GITHUB_ENV        

- name: SETUP FEATURE BRANCH
  if: ${{ github.ref != 'refs/heads/main' && github.ref != 'refs/heads/master' }}
  run: |
    echo "ENV=dev" >> $env:GITHUB_ENV        
    echo "TAG=${{ github.ref_name }}" >> $env:GITHUB_ENV
    echo "PWD=$(Get-Location)" >> $env:GITHUB_ENV        
```

Notationally, the only difference here is `>> $GITHUB_ENV` and `>> $env:GITHUB_ENV`.  Powershell requires the pre-suffix of `env:` (as in `Get-ChildItem env:`).  The consumer syntax of this env var remains the same between shells - `${{ env.TAG }}` - so it's only the creation of this env var that needs to change between shells.

For context, I've pasted below an example of where the `TAG` env var is being consumed:

```yml
- name: BUILD AND PUSH
  run: |
    try {
        cd ${{ env.ROOT_DIR }}   
        docker build -t ${{ env.ACR_NAME }}/${{ env.APP_DOCKERIMAGE }}:${{ env.TAG }} -f ${{ env.ROOT_DIR }}/${{ env.APP_DOCKERFILE }} .
        docker push ${{ env.ACR_NAME }}/${{ env.APP_DOCKERIMAGE }}:${{ env.TAG }}
    } catch {
        Write-Output "Could not push to ACR, error occured with docker build"
        Exit 1
    }        
```

# CICD

Here's a note on being sympathetic to our development teams' nuances...

{{< hint info >}}

We are using the approach of regenerating feature images and deploying from one workflow_dispatcher instead of triggering a deployment from a merge to a dedicated development branch. Our (the virtual team I'm managing that is the Platform Team - senior staff) combined experience lead us to determine that a dedicated development branch will get out of sync with our main branch.  This is especially problematic if your branching strategy predicates promoting to production via a merge to main from a dedicated development branch.  To compound this point, we often have multiple developers concurrently working on the same repo so this in itself presents inherent complexities so arriving at a CICD pipelines wasn't clear cut as teams have subtle nuances around how they build & deploy features/fixes.

{{< /hint >}}