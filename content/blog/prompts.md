---
title: Prompts
description: This post explains what prompts are in the context of GitHub Copilot Chat
date: 2025-04-18
draft: false
tags: [prompts, prompt engineering, ai, vscode]
---

## Summary

This post explains what prompts are in the context of GitHub Copilot Chat, how to create reusable prompts in the `.github/prompts` directory, and how to automatically include them in Copilot Chat using the `github.copilot.chat.codeGeneration.instructions` setting. An example and useful references are provided.

For more tips on writing effective prompts and using Copilot efficiently, see the [Best Practices for Using GitHub Copilot](#best-practices-for-using-github-copilot) section below.

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

## Automatically Including Prompts in Copilot Chat

To have Copilot Chat automatically use your custom prompts, configure the `github.copilot.chat.codeGeneration.instructions` setting in your workspace settings (e.g., `.vscode/settings.json`):

```json
{
  "github.copilot.chat.codeGeneration.instructions": [
    {
      "file": "readme.prompt.md"
    }
  ]
}
```

This tells Copilot Chat to include the content of `readme.prompt.md` as context for your requests.

## Example

Suppose you want Copilot to always follow your project’s code style. Create `.github/prompts/codestyle.prompt.md`:

```markdown
Always use 2 spaces for indentation and single quotes for strings.
```

Then, update `.vscode/settings.json`:

```json
{
  "github.copilot.chat.codeGeneration.instructions": [
    { "file": "codestyle.prompt.md" }
  ]
}
```

Now, Copilot Chat will automatically consider your code style instructions.

## References

- [GitHub Copilot Chat Cookbook](https://docs.github.com/en/copilot/copilot-chat-cookbook)
- [GitHub Copilot Chat Documentation](https://docs.github.com/en/copilot/copilot-chat)
- [Configuring Copilot Chat](https://docs.github.com/en/copilot/copilot-chat/configuring-copilot-chat)
- [Best Practices for Using GitHub Copilot](https://docs.github.com/en/copilot/using-github-copilot/best-practices-for-using-github-copilot)
