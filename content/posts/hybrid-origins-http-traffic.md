---
title: "Hybrid Origins Http Traffic"
date: 2022-01-15T17:25:37Z
draft: false
tags: [cloudflare, dns, uri forwarder, page rules, zones, domain, hostname]
---

We're migrating our on-premise workloads to Azure.  This has presented several challenges.  One of which is what I am covering specifically here in this post and that is ...

> How to reduce code change effort?

This isn't about updating runtimes, this is about having workloads spread across different platforms that need to talk to each other (with some HTTP chaining 👀).  It is not uncommon for one HTTP API to need to talk to another HTTP API.  If you move one HTTP service to run in different zone, you will need update references in those ALL dependant services so they point to that new DNS Hostname.  This isn't a big issue if you've only few HTTP APIs.  However, if you've 100+ HTTP APIs that you're migrating, including many with several HTTP dependencies, then this quickly becomes daunting and a potential PR approval and deployment (multiple environments) scheduling nightmare.  A simple domain name search in your organisation's Github account will illuminate my point.

Ok, so the nightmare scenario has been painted.  What can you do.

With the HttpClients we're using - .NET Framework and .NET Core, as a default have AllowAutoRedirect as true.  So why is this important.  Well, if you want to simply have a redirect rule return a temporary redirect - 302 - then your HttpClient will automatically react to this and reissue the same HTTP Request.  However, it does not use the original Authorization header.  Yikes.  

We use CloudFlare.  They are the best at what they do.  We're looking; as company we have an excellent relationship with CF and with that we get Stirling advice. 

We originally used their Page Rule redirect to map to a different host.  This is where we observed that the authz header being stripped on the AutoRedirect.  

Ok, so what can be do?  Well, we can make lots of additional code changes to plug this use-case in all HTTP Handler related code and be hated by all engineers for the rest of my days or look for a cross-cutting solution.  Back to CF we go... .  Two of my guiding principles for the migration effort is (1) to reduce cognitive overload and (2) effort required (this includes unnecessary code changes).  Ultimately, simplify _all_ facets of the migration.  I don't want my engineers feeling the same levels of pain that I am.

I can confirm we have a solution and it is in play.  Boo-yah!

There are few things you have to do.  Here's a short list that I will go into more detail on shortly:

- CF Page rules in domain you're mapping from
- CF CNAME set as DNS proxy
- DNS forwarders if you've dev/test environment on-premise
- If you're using Windows VMs with IIS to server up HTTP APIs, you need to a new hostname to your bindings list
- nginx origin configs (to receive and forward requests from new hostname so you can have weighted traffic load-balanced across both new and old environments during the rollout of to the new target of a particular HTTP API)

Yes, there's plenty there, most of which can be automated - think Terraform.

I'm going to focus on changes to CF here as this is where the real magic happens!
