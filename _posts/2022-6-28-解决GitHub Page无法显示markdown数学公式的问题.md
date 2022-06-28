---
layout:     post
title:      "解决GitHub Page无法显示markdown数学公式的问题"
date:       2022-6-28
author:     "tofucurd"
header-img: "img/post-bg-1.jpg"
catalog: true
mathjax: true
tags: [Others]
---

如果你使用的是Jekyll（GitHub默认），那么可以在需要的markdown文档前面的YAML内添加``mathjax:true``。

再在``_config.yml``中添加：

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

再在``_include/``下的``mathjax_support.html``（没有的话就加在``head.html``中，也有可能是其他名字，但只要意思是"head"就可以，但优先加``mathjax...``），添加：

```yml
<script type="text/x-mathjax-config">MathJax.Hub.Config({tex2jax: {inlineMath: [['$','$'], ['\\(','\\)']]}});</script>
<script type="text/javascript" async src="//cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-MML-AM_CHTML"></script>
```

这样做应该可以解决。