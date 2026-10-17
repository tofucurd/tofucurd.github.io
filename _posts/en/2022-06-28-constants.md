---
layout: post
title: "Measuring Constant Factors"
subtitle: "For reference only"
date: 2022-6-28
lang: en
ref: constants
permalink: /en/2022/06/28/constants/
tags: [Windows, macOS, Linux, gcc, clang]
---

Test environment: macOS 12.3, Apple M1 Pro.

`g++-11`: Homebrew GCC 11.3.0_1

`clang++`: Apple clang 13.1.6

### Addition, multiplication

| (0) | clang++ (ms) [O0/O2] | g++-11 (ms) [O0/O2] |
| :-- | :--: | :--: |
| 1e9 additions (no mod) | 943.698/0.002 | 548.563/0.004 |
| 1e9 additions (mod 998244353) | 5535.621/0.004 | 7795.044/0.003 |
| 1e9 multiplications (no mod) | 937.686/0.002 | 937.614/0.002 |
| 1e9 multiplications (mod 998244353) | 6496.937/0.002 | 8456.852/0.004 |
| `__int128` 1e9 (1) additions (no mod) | 1258.685/0.003 | 2827.338/0.002 |
| `__int128` 1e9 (1) multiplications (no mod) | 1311.491/0.004 | 4526.648/0.002 |

(0): every test uses a `for` loop; unless stated otherwise the variables are `long long`.

(1): range $[10^{20}, 10^{20}+10^9]$.

### Segment tree, Fenwick tree

| (2) | segment tree (ms, Ofast) [clang++/g++-11] | Fenwick tree (ms, Ofast) [clang++/g++-11] | ratio |
| :-- | :--: | :--: | :--: |
| 4e7 point updates, 4e7 range queries | 13966.705/12262.287 | 2552.754/2612.905 | 5.471:1 / 4.693:1 |

(2): both implementations are my own.

### bitset

Workload: `bitset<1e8>` XORed with itself 5000 times.

| Test | Time [clang++/g++-11] |
| :--: | :--: |
| 1 | 4209.046/7040.604 |
| 2 | 4163.929/7087.2840 |
| 3 | 4163.703/6952.518 |
| 4 | 4166.918/6908.187 |
| 5 | 4180.575/6964.1060 |
| ≈ `for` loop | 4.45e9/1.785e10 |
