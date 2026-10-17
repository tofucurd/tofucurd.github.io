---
layout: post
title: "Maximum Average Over a Subarray"
subtitle: "Slope trick, outside of DP"
date: 2022-6-29
lang: en
ref: interval-average
permalink: /en/2022/06/29/interval-average/
mathjax: true
tags: [convex-hull-trick, monotonic-queue]
---

### Problem

Given a sequence of non-negative integers, find the maximum average over subarrays whose length satisfies some condition — say, length in $[L,R]$.

The binary search solution is skipped here.

### An $O(N)$ solution

Let $S$ be the prefix sums. The task becomes: find indices $i,j$ with $0 \le j<i \le N$ and $L \le i-j \le R$ maximizing $\frac{S_i-S_j}{i-j}$. If you read index $i$ as the $X$ coordinate and $S_i$ as the $Y$ coordinate, this is choosing two eligible points so that the slope between them is maximal.

Since $S$ is non-decreasing, sweep from left to right. For each $i$, over the legal window $[i-R, i-L]$, keep a monotonic queue whose points — head to tail (the insertion end) — are monotonically decreasing in slope with respect to point $i$ (note: this is *not* a convex hull), and all of which are points that may still update the answer (note: not necessarily the point with maximum slope to $i$). Insert with a delay of $L$, pop anything older than $R$, and maintain monotonicity.

Why is the head not the point with maximum slope to $i$? Because the point with maximum slope to $i$ may already have been popped from the tail by some other point before $i$ arrives. That is entirely possible. Take these points:

$A(3,9),B(4,12),C(5,15),D(8,26),E(9,28),F(10,29)$

$A,B,C$ are inside the legal window; $D,E,F$ are the points being scanned. $A$ is added when $D$ arrives and lands at the head, $B$ is added when $E$ arrives, and $C$ is added when $F$ arrives.

When $E$ arrives, $k_{EB}$ pops $k_{EA}$; when $F$ arrives, $k_{FC}$ is placed behind $k_{FB}$, so $k_{FB}$ updates the answer. Yet the point with the largest slope to $F$ is actually $A$ — and $k_{FA}$ can never update the answer, because when $D$ was processed, $k_{DA}$ was already better than $k_{FA}$; by the same argument no later point can beat $k_{DA}$ with respect to $A$ either.
