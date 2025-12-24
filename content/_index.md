---
linkTitle: Welcome
title: Welcome
prev: /docs/guide/shortcodes/tabs
next: /docs/advanced/multi-language
layout: hextra-home

# cascade:
#   type: blog
---

👋 Hey! Welcome to my blog, where I share insights, stories, and a few extra surprises!

<!--more-->

<style>
  .home-layout {
    display: grid;
    grid-template-columns: 1fr;
    gap: 2rem;
    margin-top: 2rem;
  }
  @media (min-width: 768px) {
    .home-layout {
      grid-template-columns: 1fr 2fr;
    }
  }
</style>

<div class="home-layout">

<div>

{{< cards cols="1">}}
  {{< card link="blog" title="Blog" icon="annotation" >}}
  <!-- {{< card link="nuggets" title="Nuggets" icon="bookmark-alt" tag="new" tagType="error" >}} -->
  <!-- {{< card link="docs" title="How To" icon="translate" >}} -->
  {{< card link="ai" title="Artificial Intelligence" icon="academic-cap" >}}
  {{< card link="gaming" title="Gaming" icon="puzzle" >}}
  {{< card link="about" title="About" icon="information-circle" >}}
{{< /cards >}}

</div>

<div>

## Recent Posts

{{< recent-posts section="blog" limit="5" >}}

</div>

</div>
