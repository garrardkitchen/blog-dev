---
aliases: [/blog/hugo-shortcodes/]
title: "Hugo Shortcodes - my first try"
date: 2020-04-06T15:31:12+01:00
tags: [engineering, hugo, shortcodes, example]
---


In this article, you'll learn how Hugo shortcodes package reusable presentation without embedding repeated HTML in Markdown. That matters because durable engineering comes from understanding trade-offs, not merely reproducing a command or pattern.

# My first attempt

Here's my first effort at creating a shortcode.

This shortcode is available [here](https://github.com/garrardkitchen/blog/blob/master/layouts/shortcodes/note.html)

### Information

#### Basic

{{</* note
    Sample text
    */>}}

{{< note >}}
Sample text
{{< /note>}}

#### With italics

{{</* note italic="true"
    Sample text
    */>}}

{{< note italic="true" >}}
Sample text
{{< /note>}}

#### With header

{{</* note title="With header">
    Sample text
    */>}}

{{< note title="With header">}}
Sample text
{{< /note>}}

### Warning

#### Basic

{{</* note warning="true">
    Sample text
    */>}}

{{< note warning="true">}}
Sample text
{{< /note>}}

#### With italic

{{</* note warning="true" italic="true"
    Sample text
    */>}}

{{< note warning="true" italic="true" >}}
Sample text
{{< /note>}}

#### With header

{{</* note warning="true" title="With header"
    Sample text
    */>}}

{{< note warning="true" title="With header">}}
Sample text
{{< /note>}}

### Error

#### Basic

{{</* note error="true"
    Sample text
    */>}}

{{< note error="true">}}
Sample text
{{< /note>}}

#### With italic

{{</* note error="true" italic="true"
    Sample text
    */>}}

{{< note error="true" italic="true">}}
Sample text
{{< /note>}}

#### With header

{{</* note error="true" title="With header"
    Sample text
    */>}}

{{< note error="true" title="With header">}}
Sample text
{{< /note>}}

## Deepening the article

## Treat a shortcode as an interface

A shortcode has callers, parameters, defaults, and rendered output; changing any of them can break old content. Prefer named parameters once a shortcode has more than one meaningful input, validate required values, and fail the build with a useful message rather than silently emitting malformed HTML.

~~~go-html-template
{{ $text := .Get "text" }}
{{ if not $text }}
  {{ errorf "notice shortcode requires text: %s" .Position }}
{{ end }}
<aside class="notice notice--{{ .Get "type" | default "info" }}">
  {{ $text | markdownify }}
</aside>
~~~

Hugo templates escape values according to context. Do not mark arbitrary author input as safeHTML simply to make rendering work; that bypasses an important boundary. Decide explicitly whether a parameter accepts plain text, Markdown, or trusted HTML.

Keep presentation in the theme or asset pipeline and semantics in the shortcode. An aside with meaningful text survives a CSS failure and works better for assistive technology than a collection of decorative div elements.

## References
- [Hugo documentation](https://gohugo.io/documentation/)
- [Hugo shortcode documentation](https://gohugo.io/content-management/shortcodes/)

## Closing thought

A shortcode becomes worthwhile when it centralises semantics and safety, not merely when it saves an author from typing the same HTML twice.
