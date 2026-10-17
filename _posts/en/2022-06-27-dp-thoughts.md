---
layout: post
title: "Notes on a Class of DP Problems"
date: 2022-6-27
lang: en
ref: dp-thoughts
permalink: /en/2022/06/27/dp-thoughts/
mathjax: true
tags: [DP, SegmentTree]
---

### The problem, roughly

There are many one-dimensional number lines stacked vertically (time runs from front to back). Each line carries some intervals. The protagonist has to travel from the topmost line down to the bottom one, either required to pass through the intervals or forbidden from doing so. Horizontal movement costs something. Find the minimum cost. The value range is usually large.

### Thinking

There is of course a $dp_{i,j}$: the minimum cost of reaching position $j$ on the $i$-th line. The recurrence must be

$$dp_{i,j}=\min_{k} {\{dp_{i-1,k}+|j-k| \times cost\}}$$

where $cost$ is a constant.

The first move is to split the absolute value into $k \le j$ and $k \ge j$, maintaining a segment tree of interval minima for each case. But then the update cannot be applied to an entire interval at once.

The second move is the observation: in this setting only the interval endpoints matter, because moving horizontally first and then vertically is equivalent to moving vertically first and then horizontally.

With that, the problem collapses to range query plus point update.
