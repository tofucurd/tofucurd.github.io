---
layout: post
title: "Making Math Render on GitHub Pages"
date: 2022-6-28
lang: en
ref: gh-pages-mathjax
permalink: /en/2022/06/28/gh-pages-mathjax/
tags: [tutorial, GitHub Pages]
---

If you are using Jekyll (GitHub's default), add `mathjax: true` to the YAML front matter of the markdown document that needs it.

Then add this to `_config.yml`:

```yml
markdown: kramdown
kramdown:
  input: GFM                            # use Github Flavored Markdown !important
  syntax_highlighter_opts:
    span:
      line_numbers: false
    block:
      line_numbers: true
      start_line: 1
```

Then in `_includes/mathjax_support.html` — if it does not exist, put it in `head.html`, or whatever file plays the role of "head", but prefer the one named after mathjax — add:

```html
<script type="text/x-mathjax-config">MathJax.Hub.Config({tex2jax: {inlineMath: [['$','$'], ['\\(','\\)']]}});</script>
<script type="text/javascript" async src="//cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-MML-AM_CHTML"></script>
```

That should fix it.
