---
title: "Lerna"
date: 2020-05-04T16:59:03+01:00
tags: ["lerna", "nodejs", "twilio", "twilio sync", "twilio workers", "twilio flex", "pattern"]
draft: true
featured: true
---

## Phew, not being Furloughed

No, this isn't a posts about how to manage being furloughed or how to best to WFH (work from home) or how to dress when using Teams or Zoom!  Instead, this is a post about our, in part, journey on migrating to Twilio Flex. The focus of this particular post is on:

- how we are **propagating** domain entity state using Twilio Sync
- how we are using a **monorepo** to reduce complexity and to share common code
- how we are using **GitHub Actions** to circumvent extant CI/CD tool (Bamboo), to deploy containers to our internally maintained Docker Swarm via **Portainer**.

During the recent COVID-19 epidemic, I was 1 of 5 lucky employees who were not furloughed from a company of more than 500 employees. We were kept on, albeit, a shorter week to reimagine and improve our company's CRM system.  Our CRM, in part, is driven by the Twilio telephony API.  Sprints were created, stories were furnished and off we set, migrating over to Twilio's new Flex product.

## Guiding Principles

I set about capturing a list of guiding principles to help expedite architectural decisions and help steer our thoughts and implementation.  We are, we appropriate, using Twilio services to help with our Separation of Concerns - doing the right thing in the right place.  Where at all possible, if in the unlikely event of our internal systems experiencing a transient outage, be able to continue communicating with our applicants and customers.  As well as this, and more importantly, we didn't want to call back into our HTTP APIs to READ entity state.  To make this possible, we're using Twilio Sync to hold our domain entity state.  This state would then be accessed during Twilio Flow and Flex.  This state comes from any changes made to an entity within our CRM that is required by Flex.  An example of this is when an application status has changed or if an account manager has been allocated to an application.  So, to feed Sync, we're using a producer/consumer pattern - EDA (Event driven architecture).  And this is what this post is about, using EDA to persist state to Twilio Sync.

As with any system, when it needs to work with the `new shiny`, changes are needed.  We all know from good/bad experiences, changes to codebases can be challenging, especially when some codebases become quite britial. As a guiding principle, were we can, we use EDA.  These are some of the guiding principles, relating to state:

- Keep simple, address the constraints (think Theory of Constraints)
- No cross-domain HTTP Requests or chaining
- Use E/MDA to circumvent cross-boundary responsibility violation
- Separation of Concerns (doing the right thing in the right place)
- Avoid HTTP Request/Response pattern where possible, and use state stored in caching service (Twilio Sync) for all telephony feeds

We didn't want to make any code changes that would potentially break extant codebases and so to limit the responsibility of these changes, I devised a plan to add producers these extant codebases (either services or HTTP APIs), that would place a message on a RabbitMQ Exchange in all scenarios where state changes of an entity that is required to support our  Flex implementation.  Our RabbitMQ implementations uses MassTransit.  This incidently will make are future life & shift over to Azure less of change by simply swapping out RabbitMQ for the Service Bus provider.  If another consumer needs to consume these messages, then they too would take advantage of this loosely coupled pattern.

I promise, I'll add a diagram shortly to break up this textual content!

## Consumer Responsibilities

So, we're going to have producers to place messages on RabibtMQ exchanges (fanout configuration). This is where the consumers come in.  We're going to have initially several consumers and I envisage this to grow especially as we widen the reach and scope of Flex within our business.

### Responsibility #1

A consumer will be responsible for consuming an entity state change and forwarding this onto Twilio Sync.  This pipeline may require furnishing the state with additional information so calling out to our MSSQL is a requirement.  Some scenarios include changes to agents or to use Twilio's parlance here, Workers and so these state changes need to propagate through to Twilio Workers.

### Responsibility #2

Rebuild state via HTTP API endpoint or web portal