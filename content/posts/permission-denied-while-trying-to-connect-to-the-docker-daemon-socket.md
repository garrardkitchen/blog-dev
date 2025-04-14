---
title: "Permission Denied While Trying to Connect to the Docker Daemon Socket"
date: 2022-01-07T08:20:20Z
tags: [docker, linux, github actions, GHA, Self-Hosted Runner, dotnet, Azure Container Registry, ACR, containers, pods]
---

Out of the blue today, my first day back after Christmas break, I got this when running a GH Actions Workflow on one of our Self-Hosted Linux Runners 😱:

{{< hint danger >}}

Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get "http://%2Fvar%2Frun%2Fdocker.sock/v1.24/containers/json": dial unix /var/run/docker.sock: connect: permission denied
{{< /hint >}}

{{< hint info >}}

We have several GitHub Self-Hosted Runners running on Linux and Windows O/S that produce, amongst other artefacts, Linux and Windows images.  These images are pushed to ACR.  We're in the process of migrating our on-premise real-estate - IIS Web apps, Windows Services, Docker swarm containers - to AKS as well as migrating our SQL Server AG to Azure.  We're using Self-Hosted Runners as we have spare compute capacity and some of our applications have a dependency on a legacy NuGet server which requires our CI pipelines to run in our network.  We are in the process of also migrating these legacy packages to our Azure DevOps NuGet Feed as part of our modernization initiative. This modernization initiative encompasses upgrading our runtimes to .NET Framework 4.8 and .NET 6.0.

{{< /hint >}}

It had been running fine prior to my break so what gives?  I started to investigate...

I logged in to the Linux VM where this particular Self-Hosted Runner is hosted with the same credentials as used when I installed the Self-Hosted Runner originally.  I used the following command to confirm the same outcome:

```powershell
docker ps
```

Yup, same thing.

The next configuration I wanted to check was whether this user is a member of the `docker` group so I used this command:

```powershell
sudo groups <user>
```

Mmmmm, that's odd.  This user wasn't a member and therefore begs the question, how did this ever work in the first place?!!

I added this user using this command:

```powershell
sudo usermod -a -G docker <user>
```

I ran `docker ps` again but still no dice. 🤔.

I then checked the status of the docker service using this command:

```powershell
sudo systemctl status docker
```

It reported:

{{< hint info >}}
Active: active (running) since Thu 2021-09-16 14:13:04 UTC; 3 months 20 days ago
{{< /hint >}}

Ok, what next? 🤔

I decided to restart the self-hosted service so I entered these commands:

```powershell
cd actions-runner
sudo ./svc.sh start
```

This is when I saw these failures:

{{< hint danger >}}

Dec 19 22:01:15 *redacted* runsvc.sh[291703]: 2021-12-19 22:01:15Z: Runner connect error: The HTTP request timed out after 00:01:00.. Retrying unt…econnected.

Dec 19 22:02:35 *redacted* runsvc.sh[291703]: 2021-12-19 22:02:35Z: Runner reconnected.

Jan 06 14:42:21 *redacted* runsvc.sh[291703]: 2022-01-06 14:42:21Z: Running job: deploy

Jan 06 14:42:41 *redacted* runsvc.sh[291703]: 2022-01-06 14:42:41Z: Job deploy completed with result: Failed

Jan 06 14:46:19 *redacted* runsvc.sh[291703]: 2022-01-06 14:46:19Z: Running job: deploy

...
{{< /hint >}}

I restarted the Self-Hosted Runner using these commands:

```powershell
sudo ./svc.sh stop
sudo ./svc.sh start
```

Then I logged out and back in again to confirm docker access `docker ps` and finished off by re-running the failed GH Action Workflow.  🥳 Equilibrium is once again restored.  As per protocol, I shared issue and resolution with our IT Team in case this crops up again when I'm not online to help.