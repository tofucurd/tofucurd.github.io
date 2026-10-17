---
layout: post
title: "A Few Results Worth Keeping"
subtitle: "So I never spend an hour on them again"
date: 2022-3-16 12:00:00
lang: en
ref: conclusions
permalink: /en/2022/03/16/some-results/
mathjax: true
tags: [math, trees]
---

# 1

For $\prod_{i=1}^{+\infty}{(x^i-1)}$, let $c_i$ be the coefficient of $x^i$ after expansion. Then

$$c_i=\begin{cases}\pm 1 &\mbox{if } f(i) \in \mathbb{Z}\\ 0 &\mbox{if } f(i) \notin \mathbb{Z}\end{cases}$$

where

$$
f(x)=\frac{\sqrt{24x+1}+1}{6}
$$

# 2

For the intersection of the edge sets of two tree paths, $(a,b) \cap (c,d)$:

let $T_{xy}=\mathrm{LCA}(x,y)$, and let $x$ and $y$ be the two deepest points among $T_{ac},T_{ad},T_{bc},T_{bd}$. Then

### 1: $x=y$

The intersection is empty.

### 2: $x \neq y$

The intersection is the path $(x,y)$.
