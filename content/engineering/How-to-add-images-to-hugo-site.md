---
aliases: [/blog/How-to-add-images-to-hugo-site/]
title: "How to add images to hugo site"
date: 2020-04-06T15:31:12+01:00
draft: true
featured: true
tags: [engineering, hugo, images, paste]
---


In this article, you'll learn how to place and reference images in Hugo while keeping content portable. That matters because durable engineering comes from understanding trade-offs, not merely reproducing a command or pattern.

## Adding images to blog

For adding images to my blog through VSCode, I use an extension called Paste Image.  [Click here](https://github.com/mushanshitiancai/vscode-paste-image) for it's GitHub repos.

It comes with many configuration settings.  I've used 2 so far.  These settings enable me to (1) place the resulting image into the correct folder location and (2) to prepend the markdown link syntax so it points correctly to the image's location in my folder structure.

VSCode `settings.json`:

```json
"pasteImage.prefix": "../",
"pasteImage.path": "${currentFileDir}/img/",
```

When I press `ctrl+Alt+v` this would be injected into my markdown:

```
![](/blog/img/2020-04-06-09-56-57.png)
```

And the .png file will appear in my folder structure in the correct location:

![](/blog/img/2020-04-06-09-56-57.png)

## Deepening the article

## Choose a content model first

Hugo can serve an image from the static directory, but a page bundle is usually easier to move and process. Put index.md and its images in the same directory, then resolve the resource from the page:

~~~go-html-template
{{ with .Page.Resources.GetMatch "diagram.png" }}
  <img
    src="{{ .RelPermalink }}"
    width="{{ .Width }}"
    height="{{ .Height }}"
    alt="Diagram showing the request path">
{{ end }}
~~~

Width and height reduce layout shift. Meaningful alternative text describes the image's purpose; a decorative image should use an empty alt value. A caption belongs in a figure and figcaption, not in alt text.

Keep the original source image, but publish an appropriate format and size. Hugo image processing can resize, crop, rotate, and convert bundle resources during the build. Avoid pasting multi-megabyte screenshots directly into an article, and remove credentials, tenant names, or personal information before committing an image: blurring is often reversible enough to be risky, while cropping or replacing values is safer.

## References
- [Hugo documentation](https://gohugo.io/documentation/)
- [Hugo shortcode documentation](https://gohugo.io/content-management/shortcodes/)

## Closing thought

An image belongs in a Hugo article only when its meaning, size, privacy, and relationship to the page have been designed as carefully as its path.
