---
aliases: [/blog/prompts/]
title: Prompts
description: This post explains what prompts are in the context of GitHub Copilot Chat
date: 2025-04-18
draft: false
tags: [ai, prompts, "prompt engineering", vscode]
---


In this article, you'll learn how prompt files and repository instructions shape GitHub Copilot interactions without replacing review and verification. That matters because useful AI integration depends as much on explicit trust and verification as it does on model capability.

## Summary

This post explains what prompts are in the context of GitHub Copilot Chat, how to create reusable prompts in the `.github/prompts` directory, and how to automatically include them in Copilot Chat using the `github.copilot.chat.codeGeneration.instructions` setting. An example and useful references are provided.

For more tips on writing effective prompts and using Copilot efficiently, see the [Best Practices for Using GitHub Copilot](https://docs.github.com/en/copilot/using-github-copilot/best-practices-for-using-github-copilot).

## Understanding Prompts in GitHub Copilot Chat

Prompts are instructions or context you provide to GitHub Copilot Chat to guide its code generation or responses. Well-crafted prompts help Copilot understand your intent, resulting in more accurate and relevant suggestions.

## Creating Custom Prompts

You can create reusable prompts by adding `.prompt.md` files in your repository’s `.github/prompts` directory. Each file should contain a specific instruction or context you want Copilot to use.

**Steps:**
1. Create a `.github/prompts` directory in your repository if it doesn’t exist.
2. Add a Markdown file (e.g., `readme.prompt.md`) with your prompt content.

Example (`.github/prompts/readme.prompt.md`):
```markdown
Always use 2 spaces for indentation and single quotes for strings.
```

## Automatically Including Prompts in Copilot Chat (User or Workspace settings)

To have Copilot Chat automatically use your custom prompts, configure the `github.copilot.chat.codeGeneration.instructions` setting in your workspace settings:

```json
{
  "github.copilot.chat.codeGeneration.instructions": [
    {
      "file": "readme.prompt.md"
    }
  ]
}
```

## Automatically Include Prompts in Copilot Chat (Folder settings)

To have Copilot Chat automatically use your custom prompts, configure the `github.copilot.chat.codeGeneration.instructions` setting in your Folder settings (has highest priority and will override both User and Workspace settings):

```json {filename=".vscode/settings.json"}
{
  "github.copilot.chat.codeGeneration.instructions": [
    {
      "file": ".github/prompts/readme.prompt.md"
    }
  ]
}
```

>[!NOTE]
> You will need to include the full path for Copilot to find this file

This tells Copilot Chat to include the content of `readme.prompt.md` as context for each request.

## Example

Suppose you want Copilot to always follow your project’s code style. Create `.github/prompts/codestyle.prompt.md`:

```markdown
Always use 2 spaces for indentation and single quotes for strings.
```

Then, update `.vscode/settings.json`:

```json {filename=".vscode/settings.json"}
{
  "github.copilot.chat.codeGeneration.instructions": [
    { "file": ".github/prompts/codestyle.prompt.md" }
  ]
}
```

Now, Copilot Chat will automatically consider your code style instructions.

> [!TIP]
> To share instructions across multiple projects within a VS Code Workspace, create a **`.github/prompts/`** folder in your elected main project and start adding instruction markdown files in that folder.

## Custom instructions settings

> [!NOTE]
> There are multiple custom instructions settings.  These are listed below

- **`github.copilot.chat.codeGeneration.useInstructionFiles`**: controls whether code instructions from .github/copilot-instructions.md are added to Copilot requests.
- **`github.copilot.chat.codeGeneration.instructions (Experimental)`**: set of instructions that will be added to Copilot requests that generate code.
- **`github.copilot.chat.testGeneration.instructions (Experimental)`**: set of instructions that will be added to Copilot requests that generate tests.
- **`github.copilot.chat.reviewSelection.instructions (Preview)`**: set of instructions that will be added to Copilot requests for reviewing the current editor selection.
- **`github.copilot.chat.commitMessageGeneration.instructions (Experimental)`**: set of instructions that will be added to Copilot requests that generate commit messages.
- **`github.copilot.chat.pullRequestDescriptionGeneration.instructions (Experimental)`**: set of instructions that will be added to Copilot requests that generate pull request titles and descriptions.

## 2026 technical review

## 2026 update: prompt files are not instruction files

The settings-based code-generation and test-generation instruction mechanism described below is deprecated in current VS Code. Use file-based custom instructions instead:

- .github/copilot-instructions.md for repository-wide, always-on guidance;
- .github/instructions/*.instructions.md with applyTo patterns for file- or task-specific guidance; and
- .github/prompts/*.prompt.md for reusable prompts that a user invokes for a particular task.

A prompt file is an on-demand workflow and can declare inputs, tools, and model preferences. It is not automatically attached to every chat merely because it sits in the prompts directory. Keep instructions concise, explain non-obvious reasons, and use examples for conventions that a formatter cannot enforce.

Instructions influence model behavior; they are not policy enforcement. Back important requirements with compilation, tests, linters, security controls, and human review. Treat instructions from dependencies, copied issues, and tool output as untrusted context when they conflict with repository or user intent.

## References

- [GitHub Copilot Chat Cookbook](https://docs.github.com/en/copilot/copilot-chat-cookbook)
- [GitHub Copilot Tips and Tricks](https://code.visualstudio.com/docs/copilot/copilot-tips-and-tricks)
- [Customise Copilot Chat](https://code.visualstudio.com/docs/copilot/copilot-customization)
- [Best Practices for Using GitHub Copilot](https://docs.github.com/en/copilot/using-github-copilot/best-practices-for-using-github-copilot)
- [VS Code custom instructions](https://code.visualstudio.com/docs/agent-customization/custom-instructions)
- [VS Code prompt files](https://code.visualstudio.com/docs/agent-customization/prompt-files)

## Closing thought

A prompt file can make intent repeatable, but only tests, tools, and review can turn that repeated intent into dependable software.
