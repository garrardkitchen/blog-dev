---
title: "Shortcode - An image with a caption"
date: 2024-02-26T20:12:39Z
tags: [shortcode, caption, html, hugo, image]
---

I had an article I wanted to post but knew with the number of images I planned on including, it will look unsightly.  The first idea I had to make it less so, was to indent the image to break up the page a little.  The next idea I had was to include a caption under each image.

Knowing that HTML has an element called figure, I decided to use this.  I didn't want anything complicated so I planned to keep the code to a minimum.

When creating shortcodes, you add a file to the `📁 layouts\shortcodes\` folder.

The name of this file is important.  This name is what you used to invoke that shortcode.  I decided to call my figure.

I created this file and called it `figure.html`. 

I added my HTML. I'm using hugo expressions within the HTML to obtain the attribute values from the eventual declaration:

```xml
<figure>
    <img src="{{ .Get "src" }}" alt="{{ .Get "alt" }}">
    <figcaption>{{ .Get "caption" }}</figcaption>
</figure>
```

To add a declaration to my markdown post, I would add something similar to this:

```
{{</* figure 
    src="../img/2024-02-25-10-54-40.png" 
    alt="" 
    caption=".NET Aspire dashboard" */>}}
```

This particular example will produce this output:

{{< figure 
    src="../img/2024-02-25-10-54-40.png" 
    alt="" 
    caption=".NET Aspire dashboard" >}}

You should see both an indentation and caption.

There you have it, a short and simple example of applying an indent and a caption to an image to break the page up.  

You can see this shortcode being used here in this particular post ➡️ [.NET Aspire and Redis](../dotnet-aspire-and-redis).

I'm not sure I'm happy with it's name. On reflection, I will likely change this to `caption`.