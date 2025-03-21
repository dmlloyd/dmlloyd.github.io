---
layout: post
title: Comments enabled!
description: I've enabled comments via Giscus and GitHub.
categories:
author: dmlloyd
date: 2025-03-21 14:40:36
---

I've enabled comments on this blog using [Giscus](https://giscus.app/). It was pretty simple; here are the steps.

### Make sure your blog or site is properly prepared

Your site or blog will need to ensure the following:

* GitHub discussions must be enabled, typically with a category just for your blog/site
* The Giscus app must be installed on the organization (see below)
* The repository must be public

### Follow the instructions!

The [landing page for Giscus](https://giscus.app/) gives instructions for how to configure it for your site. Follow
these instructions; they're quite straightforward! After entering your configuration on the page, you will get
a chunk of HTML (specifically, a `<script>` tag) that you'll need to copy.

### Add a layout template

If you're like me, you are using the default blog post layout (aka `:theme/post`).
In order to add comments, you need to customize this layout a bit.
I added a file to my project called `templates/layouts/post.html`.
The content of that file will look something like this:

```html
---
layout: :theme/post
---

\{@io.quarkiverse.roq.frontmatter.runtime.model.DocumentPage page}

\{#insert /}

\{#article-end}
<div class="giscus"></div>
<script blah blah blah>
</script>
\{/}
```

Paste your `<script>` tag over the one in the snippet above.

### Update your posts

If you already had a customized layout for posts, you don't need to do this step.
Otherwise, if your post layout is set to `layout: :theme/post`, you'll need to go into each post's front matter
and replace the layout with `layout: post`.

That's pretty much all I had to do.
But I might tinker with things in the future; feel free to peruse [the repository for this blog](https://github.com/dmlloyd/dmlloyd.github.io)
to see how I've done things - either for your own reference, or to tell me what I've done wrong!