---
layout: post
title: "Notes on a Codeforces Blog"
subtitle: "A handful of DP techniques"
date: 2022-7-14
lang: en
ref: cf-blog
permalink: /en/2022/07/14/cf-blog-notes/
mathjax: true
tags: [DP]
---

[Original Codeforces blog](https://codeforces.com/blog/entry/47764)

[Luogu problem list](https://www.luogu.com.cn/training/189903#problems)

## Before we start

Read the summary, then solve the problems, then come back to the individual solutions.

# Bitmask DP

### Summary

Skipping the basics (~~and since I can't do FWT, CF622C has been cut~~).

### [CF453B](http://codeforces.com/contest/453/problem/B)

The $b$'s are pairwise coprime, so their prime factors cannot overlap. Since $\sum{\|a_i-b_i\|}$ must be minimized, $b_i \le a_i \times 2 = 60$, so every prime factor of a $b$ is at most $60$ — only $17$ of them. Bitmask DP: $dp_{i,s}$ is the minimum cost for the first $i$ elements of $b$ having covered the prime set $s$.

### [CF678E](http://codeforces.com/contest/678/problem/E)

The initial state is not fixed, so work backwards from the end: $dp_{i,s}$ is the probability that player $1$ survives to the end, given that the living set is $s$ and $i$ survived the previous round. The transition enumerates who $i$ duels next and whether $i$ survives, taking the max over all possibilities.

# Fewer states

### Summary

Sometimes a DP that looks brute-force does not actually fill some dimension, and that can be optimized away.

### [CF505C](http://codeforces.com/contest/505/problem/C)

At first glance $dp_{i,j}$ — landing on $i$ after a jump of length $j$ — looks $O(N^2)$. But even starting from $d=1$, the jump length stays $O(\sqrt{N})$, so the effective range of $j$ is only $[d-400, d+400]$, which gives $O(N\sqrt{N})$.

# Total minus invalid

### Summary

The original calls this "change the object to dp", but I think it is a special case of reducing states, wrapped around the classic "total minus invalid" idea, so it gets its own section.

No separate problems, since only [CF559C](https://codeforces.com/contest/559/problem/C) and [CF722E](https://codeforces.com/contest/722/problem/E) seem to use it so far.

Sketch: on an $H \times W$ board ($H,W$ around $10^6$) there are $N$ special cells ($N \le 2000$). Count paths from the top-left corner to the bottom-right corner passing through exactly $m$ ($m \le 25$) special cells.

Recipe:

Sort the special cells by $X$, then by $Y$, and add $(H,W)$ to the set; now it is "exactly $m+1$ special cells".

$dp_{i,j,k}$ — reaching $(i,j)$ having passed $k$ special cells — is obviously hopeless. But the DP value only actually changes at special cells, so only transitions between special cells matter.

The simplified version, $m=0$:

> Let $dp_i$ be the number of paths from $(1,1)$ to the $i$-th special cell such that it is the first special cell on the path. The transition is total minus invalid, enumerating the first special cell visited:
>
> $$dp_i=\binom{X_i+Y_i-2}{X_i-1}-\sum_{X_j \le X_i,\,Y_j \le Y_i}{dp_j \times \binom{X_i+Y_i-X_j-Y_j}{X_i-X_j}}$$

For $m \ge 0$, define $dp_{i,j}$ as the number of ways to reach special cell $i$ having passed **at least** $j$ special cells; the transition enumerates the special cell at which exactly $j$ is reached, and the answer is a difference.

# Open and Close Interval Trick

### Summary

Could not think of a short Chinese name for it, so the original name stays.

Sketch: given a sequence of length $N$, choose some intervals (a fixed number, or any number) such that no two share a left endpoint or a right endpoint ($l_i \ne l_j$ and $r_i \ne r_j$). Count the ways (there may be extra constraints, but the idea is the same).

Recipe (any number of intervals):

Handle all intervals together and sweep left to right. When the construction reaches $i$, every interval is in one of three states:

- finished ($r \le i$), called `Closed`;
- started but not finished ($l \le i < r$), called `Opened`;
- not started yet ($i < l$).

Only the first two matter at $i$, so define $dp_{i,j}$ as the number of ways for the first $i$ positions with $j$ intervals still `Opened`.

Moving from $i-1$ to $i$, there are only five possible operations:

- do nothing, so every `Opened` interval extends by 1: $dp_{i,j} \mathrel{+}= dp_{i-1,j}$
- open an `Opened` interval: $dp_{i,j} \mathrel{+}= dp_{i-1,j-1}$
- close an `Opened` interval: $dp_{i,j} \mathrel{+}= dp_{i-1,j+1} \times j$
- open and immediately close a `Closed` interval: $dp_{i,j} \mathrel{+}= dp_{i-1,j}$
- close one `Opened` interval and open another: $dp_{i,j} \mathrel{+}= dp_{i-1,j-1} \times j$

Only one open or one close per step, because endpoints may not collide.

### [CF367E](https://codeforces.com/contest/367/problem/E)

Because no interval contains another, when closing an `Opened` interval only the earliest-opened one is eligible — no coefficient — and a `Closed` interval may not be opened. The constraint on $x$ is checked during the transition, forcing the open. For a fixed $n$, add one more dimension $k$ on top of $dp_{i,j}$.

### [CF466D](https://codeforces.com/contest/466/problem/D)

Every element's increment is fixed; call it $d$. So at each $i$ only $dp_{i,d}$ and $dp_{i,d-1}$ are meaningful, giving $O(N)$.

### [CF626F](https://codeforces.com/contest/626/problem/F)

Sort the sequence first. The cost of a group is its largest index minus its smallest index, which is just the sum of consecutive gaps across its span. So the problem becomes the one sketched above, with the extra requirement that the sum over all intervals is at most $k$.

State: $dp_{i,j,k}$ — first $i$ elements, $j$ `Opened` intervals, total $k$ so far. Since each step's gap is fixed, the third coordinate of the next state is determined: $k + j \times \Delta$. Each number belongs to exactly one group, so it cannot be opened and closed at the same time; that leaves four transitions.

With $l=k+j \times (a_{i+1}-a_i)$ and $v=dp_{i,j,k}$:

- join an `Opened` interval and keep it open: $dp_{i+1,j,l} \mathrel{+}= v \times j$
- open an interval: $dp_{i+1,j+1,l} \mathrel{+}= v$
- close an interval: $dp_{i+1,j-1,l} \mathrel{+}= v \times j$
- open a `Closed` interval: $dp_{i+1,j,k} \mathrel{+}= v$

The answer is $\sum_{i=0}^{k}{dp_{n,0,i}}$.

# Connected-segment DP

### Summary

Problem: construct a permutation satisfying some property, or carrying some cost, where the property or cost usually involves the sizes or the differences of adjacent elements. Count the solutions, or optimize.

Recipe:

Insert $i$ from small to large. After inserting $i$, the $i$ numbers form some connected segments (say $j$ of them). We do not care what each segment contains, nor how far apart the segments are. "Connected segment" here is a promise that no number may later be inserted into the middle of a segment, and that adjacent segments are not touching.

The basic state is $dp_{i,j}$ — the first $i$ numbers split into $j$ segments. There are three transitions:

- start a new segment, possibly in a gap: $dp_{i-1,j-1} \to dp_{i,j}$
- extend an existing segment on either side: $dp_{i-1,j} \to dp_{i,j}$
- join two adjacent segments through $i$: $dp_{i-1,j+1} \to dp_{i,j}$

### [P5999](https://www.luogu.com.cn/problem/P5999)

The final permutation may not contain three consecutive ascending or descending elements, so extending on either side is forbidden — only starting a new segment or joining two.

### [CF1515E](https://codeforces.com/contest/1515/problem/E)

Again three cases, but there are more of them:

- when extending an existing segment, one may skip a position, which opens the skipped one automatically;
- when joining two adjacent segments, one cannot transition to $i+1$ (had it been a single gap, it would already have opened), only to $i+2$ or $i+3$ — mind the coefficients.

### [CF704B](https://codeforces.com/contest/704/problem/B)

Inserting in increasing order means every segment already formed is smaller than $i$, so the contribution of $i$ splits cleanly between what came before and what comes after: the interaction between $i$ and earlier numbers is accounted for when $i$ is placed, and the interaction between $i$ and later numbers is charged early, at $i$. Each number's contribution is then independent.

The start/end constraints are a simplification: special-case $s$ and $e$. Note that $s$ and $e$ have no left or right contribution.

### [LOJ2743](https://loj.ac/p/2743)

Unlike [CF704B](https://codeforces.com/contest/704/problem/B), $s$ and $e$ are not given, so two extra dimensions record whether the start and the end have been covered, plus one more for the running sum. The transitions are similar to the previous problem. Tedious to implement; mind the details.
