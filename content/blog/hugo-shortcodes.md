---
title: "Hugo Shortcodes - my first try"
date: 2020-04-06T15:31:12+01:00
tags: [hugo, shortcodes, example]
---

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

