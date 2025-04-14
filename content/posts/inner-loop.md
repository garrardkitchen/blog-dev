---
title: "What is the Inner Loop?"
date: 2024-07-06T08:18:24+01:00
tags: [ci, continuous integration, clean code, code quality, Inner-loop]
draft: true
---

# Related articles
- [Inner Loop with Continuous Integration](/posts/inner-loop-ci)
- [Inner loop with Code Quality with Qodana](/posts/inner-loop-qodana-vscode)
- [Inner Loop with .NET Aspire](/posts/inner-loop-dotnet-aspire)
- [Transition to the Cloud](/posts/inner-loop-cloud-transition)

---

# What is the Inner-loop?

The "Inner Loop" in software development refers to the set of activities and processes a developer repeatedly goes through while writing, testing, and debugging code in the development phase, before committing changes to a version control system. This loop typically includes writing code, compiling, running, testing locally, and debugging. The goal of optimizing the inner loop is to maximize developer productivity and satisfaction by making these activities as efficient and frictionless as possible.

As described by Gene Kim in his book called, "DevOps Handbook", *the 2nd Way*, you optimize for fast feedback (from right to left). Building and testing locally, does contribute towards this 2nd way but we can do more, and often do, to provide additional feedback.  I will expand on this shortly.

---

# Why is this important?

I've summarized the importance of the inner loop here:

- Maximizing inner-loop time boosts productivity and personal satisfaction for developers.
- Fix bugs before integrating your changes into mainline branch.
- Reducing outer-loop friction (through better tooling and automation) to minimize disruptions.

I will now expand in the above list.

1 - *Maximizing inner-loop time boosts productivity and personal satisfaction for developers*

...

2 - *Fix bugs before integrating your changes into mainline branch.*


...

3 - *Reducing outer-loop friction*

...

---

# How do can provide more feedback?

As illuded to at the end of the opening section, there are more ways you can provide feedback

- SAST
- Analysers
- Standards
- CI feature pipeline