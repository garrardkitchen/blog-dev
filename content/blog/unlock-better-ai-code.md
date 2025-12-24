---
title: "Unlock Better AI Code - Why Small Batches are Your Secret Weapon"
date: 2025-12-24T10:00:00Z
draft: false
author: "Garrard Kitchen"
tags: ["ai-assisted-development", "prompt-engineering", "software-engineering", "best-practices", "agile"]
categories: ["Development", "Security", "DevOps"]
summary: "Discover how breaking down AI-assisted development into small, focused batches—aligned with the Single Responsibility Principle—produces better code, reduces costs, and keeps developers meaningfully engaged in the software creation process."
---

# Stop fighting your AI coding assistant

## Introduction
 
We're all learning. None of us know it all. We have opinions, we have experiences, we conduct research, we scratch the occasional itch. We watch YouTube videos and read a blog posts and think we've mastered the infinite detail of the "new shiny." We haven't though. We need real experiences for authentic opinions.
 
With this in mind and not wanting to suggest I'm a master of all things AI related, I've recently discovered spec-driven AI development, and it's changing how I think about working with AI coding assistants.
 
We've all been there - working within a context window for what feels like days, dipping in when we can, steering the AI to do what we want it to do, becoming increasingly frustrated with the output it generates. Well, if you were given a short, possibly terse requirement, you'd struggle too!
 
## The Power of Small Batches
 
What I'm learning right now is that it's all about batch size. And this batch size has to be small. When it's small, it's focused. Plus, this aligns perfectly with commit/PR sizes too. This also aligns nicely with what my own research has determined: an agent's scope must be small. And when I think small, I think of the Single Responsibility Principle (SRP).
 
Oh yeah, and costs. How many of us, if we have to pay for tokens or credits ourselves, watch our usage like a hawk? I know I do!
 
## A New Engineering Flow
 
I believe this represents a new engineering workflow emerging:
 
### Step 1: Requirements to Plan
 
Get the AI to produce a plan from your requirements. This high-level overview sets the direction without getting lost in implementation details.
 
### Step 2: Plan to Tasks
 
From the plan, have the AI produce a series of tasks (grouped). These tasks are the small batch sizes—small enough to avoid overwhelming the AI and appropriately sized to produce meaningful commits/PRs from.
 
As a side - One sentence I've personally been adding to my prompts for several months now is: *"Please create a plan before you make code changes and ask me any questions you have to help guide you."*. This simple addition has dramatically improved the quality and relevance of AI-generated code in my projects (but not perfect, just improved).
 
## Why This Matters
 
This approach offers several compelling advantages:
 
**Clarity and Focus**: Small, well-defined tasks reduce ambiguity and help the AI generate more accurate code.
 
**Cost Efficiency**: Smaller context windows mean fewer tokens consumed per interaction, making AI assistance more economical.
 
**Better Version Control**: Task-sized batches naturally align with atomic commits, improving your Git history and making code reviews more manageable.
 
**Alignment with Best Practices**: The Single Responsibility Principle isn't just for classes anymore—it applies to how we scope AI tasks.
 
## Toward Intentional AI Collaboration
 
In conclusion (thus far in my journey)…
 
We stand at the threshold of a fundamental shift in how software is created. The next stage of the agentic revolution will be defined not by the AI models themselves, but by our sophistication in collaborating with them. Just as previous generations learned design patterns, version control, and agile methodologies, we must now master the art [and it is an art!] of prompt engineering, context management, and AI task orchestration.
 
To steer this transformation for the greater good, we need a collective commitment to intentionality. This means rejecting the "move fast and break things" mentality when it comes to AI-generated code in production systems. It means developing peer review processes that account for AI-assisted development. It means training the next generation of developers not just in coding, but in critical evaluation of AI outputs.
 
Most importantly, we must preserve and celebrate what makes us uniquely human in this process: our ability to understand user needs, to make ethical trade-offs, to see the bigger picture beyond any single function or module. The small-batch, spec-driven approach isn't just a technical optimization - it's a philosophy that keeps humans in the loop at every meaningful decision point.
 
If we approach this revolution thoughtfully, with humility about what we don't yet know and commitment to what we value as engineers and humans, we can create a future where AI amplifies our capabilities without diminishing our agency. The question isn't whether AI will change how we build software - it will. The question is whether we'll shape that change deliberately, with our values and humanity intact.
