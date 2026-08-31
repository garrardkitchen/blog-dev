# Garrard Kitchen blog, insights, stories, and a few extra surprises!

## Contents

- [Folder Structure](#folder-structure)
- [Getting Started](#getting-started)
- [Adding a New Blog Post with an Image](#adding-a-new-blog-post-with-an-image)
- [Adding a Main Menu in hugo.yaml](#adding-a-main-menu-in-hugoyaml)
- [Automated README Prompting with Copilot Chat](#automated-readme-prompting-with-copilot-chat)
- [Troubleshooting](#troubleshooting)
- [Summary](#summary)

---
This repository contains the source code for the Garrard Kitchen blog, built with [Hugo](https://gohugo.io/) and the [Hextra theme](https://imfing.github.io/hextra/). Content is organized in the `content/blog` folder for blog posts and `content/ai` for AI-related articles. The project uses the Hextra theme as a git submodule for easy updates and separation of theme code.

---
## Folder Structure

```plaintext
blog-dev/
├── config.toml / hugo.yaml
├── content/
│   ├── ai/
│   ├── blog/
│   ├── about/
│   ├── docs/
│   ├── gaming/
│   └── nuggets/
├── static/
│   └── images/
├── themes/
│   └── hextra/        # Hextra theme as a git submodule
├── README.md
└── ...
```

---

## Getting Started

### 1. Clone the Repository

```sh
git clone --recurse-submodules <repo-url>
cd blog-dev
```

### 2. Install Hugo Extended (Windows)

```sh
winget install -s winget Hugo.Hugo.Extended
```

Or using Chocolatey:

```sh
choco install hugo -confirm
choco install hugo-extended -confirm
```

### 3. Update Submodules (if needed)

```sh
git submodule update --init --recursive
```

### 4. Run the Development Server

```sh
hugo server
```

Visit [http://localhost:1313](http://localhost:1313) to view your site.

---

## Adding a New Blog Post with an Image

1. **Create a new post:**

   ```sh
   hugo new blog/my-new-post.md
   ```

2. **Edit your post** in `content/blog/my-new-post.md`.

3. **Add an image:**
   - Place your image in `static/images/`.
   - Reference it in your markdown:
     ```markdown
     ![Alt text](/images/my-image.jpg)
     ```

4. **Commit your changes:**

   ```sh
   git add content/blog/my-new-post.md static/images/my-image.jpg
   git commit -m "Add new blog post with image"
   git push
   ```

---

## Adding a Main Menu in `hugo.yaml`

Edit `hugo.yaml` or `config.toml` to add menu items:

```yaml
menu:
  main:
    - name: Home
      url: /
      weight: 1
    - name: Blog
      url: /blog/
      weight: 2
    - name: AI
      url: /ai/
      weight: 3
```

---

## Automated README Prompting with Copilot Chat

This project uses a custom prompt file, `readme.prompt.md`, to guide and automate the generation and updating of this `README.md` file. The prompt is included in your VS Code user settings using the following configuration:

```jsonc
"github.copilot.chat.codeGeneration.instructions": [
    {
        "file": "readme.prompt.md"
    }
],
```

This setting tells GitHub Copilot Chat to use the contents of `readme.prompt.md` as context and instruction when generating or updating the README. This ensures that documentation remains consistent, up-to-date, and tailored to your project's needs.

- The prompt file is located at `.github/workflows/prompts/readme.prompt.md`.
- You can edit this file to change the style, structure, or required sections for your README.
- Any time you ask Copilot Chat to update the README, it will use this prompt for guidance.

---

## Troubleshooting

If you see an error like this after running `hugo server`:


> [!CAUTION]
> Error: error building site: assemble: ".../error-NETSDK1045.md:11:1": failed to extract shortcode: template for shortcode "hint" not found


Run:

```sh
git submodule update --init --recursive
```

---

## Summary

This repository provides a modular, maintainable setup for a Hugo-based blog using the Hextra theme, supporting easy content creation, image management, and menu customization.