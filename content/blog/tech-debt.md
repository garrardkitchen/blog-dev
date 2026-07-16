---
title: "Technical Debt Is a Financing Decision"
date: 2020-01-02T11:49:14+01:00
draft: true
featured: false
tags: [engineering, technical-debt, architecture, delivery]
---

In this article, you'll learn how to describe technical debt as a deliberate trade-off with interest, ownership, and a repayment strategy. That matters because durable engineering comes from understanding trade-offs, not merely reproducing a fashionable metaphor.

Technical debt is the future cost created when we choose a design that is expedient now but makes later change harder. It is not simply old code, untidy code, or every defect in the backlog. A mature team names the compromise, records why it is acceptable, and decides what evidence will trigger repayment.

## Principal, interest, and risk

The principal is the work needed to reach the more sustainable design. Interest is the recurring friction paid until then: slower changes, additional tests, operational toil, incidents, or cognitive load. Risk is the possibility that the compromise fails before the team repays it.

That framing produces better conversations than a single “tech debt” total. A small shortcut on an isolated internal tool may be rational. The same shortcut in identity, payments, or a shared deployment path can compound quickly because its blast radius and rate of change are larger.

## Make debt visible

A useful debt record should capture:

- the decision and the constraint that made it reasonable;
- affected systems and likely failure modes;
- the continuing cost, measured where possible;
- an owner and a review date;
- a repayment or containment option; and
- the event that changes its priority, such as traffic growth or a new compliance need.

Do not promise that every debt item will be removed. Some can be refinanced by isolating an old component, adding an adapter, improving observability, or reducing its change rate. Others disappear when a product is retired. The goal is informed optionality, not architectural purity.

## Budget repayment into delivery

Debt work competes with feature work only when its impact is hidden. Connect it to lead time, escaped defects, recovery time, support load, or security exposure. Then repay in small slices alongside related changes. A heroic “clean-up quarter” often creates a second queue of assumptions before the first one has been tested.

The most dangerous debt is accidental: no one remembers the trade-off, so each new engineer treats the constraint as an immutable fact. An architecture decision record, a focused test, and a clear boundary can sometimes reduce more interest than a rewrite.

## Closing thought

Technical debt is not evidence that engineers failed to predict the future; unmanaged debt is evidence that the organisation stopped revisiting the price of its past decisions.
