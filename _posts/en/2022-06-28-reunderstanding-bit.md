---
layout: post
title: "Understanding Fenwick Trees, Again"
subtitle: "How it works, and one use"
date: 2022-6-28
lang: en
permalink: /en/2022/06/28/reunderstanding-bit/
ref: bit
mathjax: true
tags: [BIT]
---

## How it works

Let $C$ be the array a Fenwick tree maintains, and $A$ the original array. Then

$$C_i=\sum_{j=i-\mathrm{lowbit}(i)+1}^{i}{A_j}$$

So a prefix sum is computed by adding $C_i$ and jumping to $i-\mathrm{lowbit}(i)$, which splits the prefix into $O(\log N)$ intervals.

A point update just adds to every interval that covers $i$. This relies on one property: the first interval after $i$ that covers $i$ is $i+\mathrm{lowbit}(i)$ — which is the clever part of the structure.

## One use

Beyond point updates and prefix queries, the interesting case is finding the global $k$-th smallest element with a Fenwick tree over the value domain.

Binary lifting: find the largest $base$ such that $(\sum_{i=1}^{base}{A_i}) < k$; then $base+1$ is the $k$-th smallest.

Since $C_i$ covers an interval of length $\mathrm{lowbit}(i)$, enumerating bits from high to low means that the currently accumulated $base+2^i$ covers exactly the last $2^i$ candidates. Code below (finds the $k$-th smallest in $[1,n]$):

```cpp
int kth(int k){
    int base=0,sum=0;
    for(int i=log2(n);~i;i--){
        base+=1<<i;                       // try adding 2^i
        if(base>=n||sum+C[base]>=k)base-=1<<i;
        else sum+=C[base];
    }
    return base+1;
}
```
