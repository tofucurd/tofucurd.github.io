---
layout: post
title: "Lowering LLVM IR to a 16-Bit ISA"
subtitle: "Three decisions the target forced"
date: 2026-10-18
lang: en
ref: lc3-design
permalink: /en/2026/10/18/lowering-llvm-ir-to-lc3/
mathjax: true
tags: [LLVM, compilers, LC-3]
published: false
---

<!--
Draft outline (delete this comment when done):

1. Phi elimination without edge splitting
   - R7 doubles as the return address and the predecessor selector
   - what this buys, what it costs

2. Register allocation with eight registers, four of them pinned
   - R4 globals base, R5 frame pointer, R6 stack pointer, R7 return address
   - only R1-R3 left for computation; spill strategy and its measured cost

3. The 32-local limit
   - LDR/STR have a 6-bit immediate offset, so the frame layout is constrained
   - how the constraint propagated into the calling convention

Tone: constraint -> tradeoff -> result. Numbers > adjectives.
Set published: true when ready.
-->
