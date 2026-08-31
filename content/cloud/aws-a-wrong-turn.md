---
aliases: [/blog/aws-a-wrong-turn/]
title: "AWS Was a Wrong Turn—for This Engagement"
date: 2022-01-31T08:29:36Z
tags: [cloud, aws, cloud-strategy, operating-model, migration]
draft: true
---

In this article, you'll learn how to separate a difficult cloud engagement from the capability of the platform, and how to test a proposed migration against business constraints before momentum makes it irreversible. That matters because a technically credible destination can still be the wrong next move.

Several years ago, a small company I worked with shifted attention from Azure to AWS following external advice. The experiment did not create the expected path to market. Time went into AMI design, nested Quick Starts, VPC integration, partner coordination, and a proposed data-streaming architecture while the product and its revenue remained delayed.

It would be easy to turn that experience into “AWS was the problem.” That conclusion would be emotionally satisfying and technically weak. AWS is a broad, capable platform. The failure was a poor fit between the proposed journey, the company's existing assets, its size, the available support, and the outcome it needed soonest.

## The decision we should have framed

The primary question was not “Which cloud is better?” It was “What is the shortest safe route to validating the product?” We already had working knowledge and assets in Azure. A smaller experiment could have preserved that progress and tested only the uncertain boundary—for example, synchronising the necessary data or placing one capability on AWS—before committing to a wider platform shift.

The proposed solution increased several forms of work at once:

- a new identity, networking, and governance model;
- unfamiliar infrastructure artefacts and deployment paths;
- new operational skills and incident procedures;
- coordination with people whose availability we did not control; and
- architectural decisions made before product demand had validated them.

None of those costs is uniquely AWS. Every cloud transition has a learning curve and an operating-model cost. The mistake was allowing platform possibility to outrun the company's capacity to absorb change.

## Treat external advice as an input

A cloud provider, partner, or consultant sees the problem through its own expertise and incentives. That does not make the advice bad, but it means the organisation receiving it must retain architectural accountability.

Before accepting a large recommendation, ask for:

1. the business outcome and the date by which it must be tested;
2. assumptions about skills, support, traffic, compliance, and budget;
3. the smallest reversible experiment that can invalidate the proposal;
4. a dependency map showing who must respond and on what timescale;
5. exit criteria, including the cost of returning to the current path; and
6. an operating model for security, observability, patching, recovery, and ownership.

Use the provider's Well-Architected guidance to examine the design, but do not mistake a framework review for product validation. A perfectly operated system can still be an expensive answer to the wrong question.

## What I would do differently

I would preserve the working Azure path, isolate the cross-cloud requirement behind a clear contract, and time-box an AWS proof of value. The proof would include deployment, identity, data flow, monitoring, failure recovery, and a cost estimate—not only a successful demo.

I would also put delay on the decision record. Architecture often models infrastructure cost while treating coordination and lost market time as free. For a small company, a month spent waiting for a dependency can matter more than a theoretical saving available after scale.

Finally, I would separate the retrospective from blame. The useful output is a better decision process: smaller commitments, explicit assumptions, named owners, and earlier evidence. The people involved were operating with the information and incentives available at the time.

## References

- [AWS Well-Architected Framework](https://docs.aws.amazon.com/wellarchitected/latest/framework/welcome.html)
- [AWS Cloud Adoption Framework](https://docs.aws.amazon.com/whitepapers/latest/overview-aws-cloud-adoption-framework/welcome.html)
- [Azure Cloud Adoption Framework](https://learn.microsoft.com/azure/cloud-adoption-framework/)

## Closing thought

A difficult AWS experience is useful evidence about an engagement and operating model, but it becomes misleading the moment it is mistaken for a verdict on an entire cloud.