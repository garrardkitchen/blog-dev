---
aliases: [/blog/twilio-serverless/]
title: "Twilio Serverless"
date: 2020-04-06T17:26:10+01:00
draft: true
tags: [cloud, twilio, serverless, studio-flow, taskrouter, "hunt groups"]
---


In this article, you'll learn how Twilio Functions fit into a serverless design, including local testing, deployment, and request validation. That distinction matters because cloud failures usually emerge at the seams between configuration, identity, networking, and operations.

Since the new year, I have been working on a new feature for our enterprise CRM system.  This new feature is called Hunt Groups.

The goal of Hunt Groups is to include a single agent (Account Manager) as a participant to a conference, when they accept the incoming Task Reservation. However, the challenge, is to offer that participant slot to potentially many agents.  In this feature, a Hunt Group is a tier of agents. There can be multiple tiers.  Each tier will be offer the Task Reservation for a specific amount of time.  If no agent from that tier accepts the Task Reservation, the Task Reservation will drop down the next tier of agents.

A diagram will help:

![](/blog/img/2020-04-06-20-37-54.png)

_show 2 tiers, 1st tier showing rejected and ignored agents, these ignored agent being include in the following tier too_

More complexity arises when, this an agent is currently part of an extant Hunt Group.  A Hunt Group will only last for the duration of the Workflow and that workflow will be deleted as soon as an agent accept the Task Reservation.

Further complexity arises when, an agent from a higher tier has not rejected the Task Reservation.  A reason for this that they are busy or away from their desk temporarily.  When this is the case, this does not mean this agent should never be re-presented with the Task Reservation.  Think of it this way; if you have 2 agents in tier A and 2 agents in tier B, if agent 1 from tier A rejects the Task Reservation and agent 2 is away from their desk, when the Task Reservation falls thru the next routing Step (which means Tier 2), this Tier will send the Task Reservation to 3 agents (2 from Tier 2 and 1 from Tier 1).

Hopefully the scene has been set!

Time for some guiding principles to helps us along the way..

TODO:
- Look thru emails from twilio
- Mention new Flex project
- Mention Conversations PoC


### Separation of Concerns


![](/blog/img/2020-04-06-20-34-40.png)



### Local Serverless testing

### Serverless deploy (GitHub Action)

#### Access code from other files
#### Save raw Twilio Event JSON to Azure Blob Storage for serverless ETL -> Data Lake

### Secure Twilio HTTP Endpoints


#### File name change
#### SHA1 hash and HMAC (Hash-based message authentication code)

## References
- [Twilio Functions documentation](https://www.twilio.com/docs/serverless/functions-assets/functions)
- [Twilio request validation](https://www.twilio.com/docs/usage/security#validating-requests)

## Closing thought

Twilio Functions reduce the infrastructure needed to handle an event, but request validation, dependency ownership, observability, and failure behavior remain part of the application.
