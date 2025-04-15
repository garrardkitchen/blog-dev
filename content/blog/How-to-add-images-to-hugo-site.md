---
title: "How to add images to hugo site"
date: 2020-04-06T15:31:12+01:00
draft: true
featured: true
tags: [hugo, images, paste]
---

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
![](../img/2020-04-06-09-56-57.png)
```

And the .png file will appear in my folder structure in the correct location:

![](../img/2020-04-06-09-56-57.png)

