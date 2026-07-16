---
title: "What Is Good Engineering?"
date: 2020-01-27T11:49:14+01:00
draft: true
featured: false
tags: [engineering, good-engineering, principles, practices, leadership]
---

In this article, you'll learn how this series defines good engineering through outcomes, principles, practices, culture, and leadership. That matters because a team can follow every fashionable technique and still fail to build the right thing safely.

Good engineering is the repeatable ability to turn uncertain needs into valuable, operable systems while preserving the capacity to change them. Code quality is part of that definition, but so are judgement, collaboration, delivery, security, cost, and the effect on people who operate or use the result.

## Principles guide judgement

[Engineering principles](/blog/principles/) give teams a vocabulary for trade-offs: simplicity, separation of concerns, cohesive design, readable code, testing, and avoiding work whose need has not been demonstrated. They are heuristics rather than laws. DRY can reduce inconsistent knowledge, but premature abstraction can couple things that only look alike. Performance work needs measurements, while security and data-loss risks may justify prevention before an incident supplies them.

The value of a principle is not that it ends a discussion. It helps a team explain why a choice fits this context.

## Practices produce evidence

[Engineering practices](/blog/practices/) turn intent into repeatable evidence. Version control records change. Small reviews expose assumptions. Automated tests assess behavior at several boundaries. Continuous delivery makes a release routine. Telemetry shows what happened after release. Threat modelling, dependency maintenance, backups, recovery exercises, and incident learning address risks that a unit test cannot.

Automation should remove repeatable error and shorten feedback. It should not make a decision opaque. A pipeline nobody can diagnose is merely manual work encoded somewhere less accessible.

## Culture determines when truth arrives

[Culture and collaboration](/blog/culture-and-collaboration/) decide whether weak signals surface while they are still inexpensive. Engineers need to challenge a plan, disclose an error, and ask for help without performing certainty. Psychological safety coexists with accountability: the standard remains high, while the response to failure examines both individual choices and the system that shaped them.

An [agile operating model](/blog/agile/) helps when it reduces batch size, exposes learning, and lets plans change as evidence arrives. Ceremonies without feedback are scheduling theatre.

## Leadership makes quality scale

[Technical leadership](/blog/leadership/) creates clarity and boundaries instead of a queue of approvals. Leaders connect engineering work to outcomes, make constraints explicit, bring the right perspectives into decisions, and record when a decision should be revisited. They develop other decision-makers and leave systems more understandable than they found them.

## A practical test

Ask five questions of a change:

1. Does it solve a real problem for a named user or operator?
2. What assumptions and failure modes are we accepting?
3. What evidence will tell us it works in production?
4. Can another engineer understand, change, and recover it?
5. Is the ongoing value worth its security, operational, financial, and cognitive cost?

There is no permanent score for good engineering. There is only the quality of the next decision and the feedback that improves the one after it.

## Closing thought

Good engineering is not the absence of compromise; it is the ability to make each compromise visible, testable, reversible where possible, and humane.
