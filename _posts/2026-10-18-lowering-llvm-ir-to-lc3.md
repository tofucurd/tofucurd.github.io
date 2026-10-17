---
layout: post
title: "把 LLVM IR 降到 16 位指令集"
subtitle: "目标机器逼出来的三个决定"
date: 2026-10-18
lang: zh
ref: lc3-design
permalink: /2026/10/18/lowering-llvm-ir-to-lc3/
mathjax: true
tags: [LLVM, 编译器, LC-3]
published: false
---

<!--
大纲（写完删掉这段注释）：

1. 不做 edge splitting 的 phi 消除
   - R7 同时是返回地址和 predecessor selector
   - 换来了什么，代价是什么

2. 八个寄存器，四个被钉死
   - R4 全局基址、R5 帧指针、R6 栈指针、R7 返回地址
   - 只剩 R1–R3 可用；溢出策略和它的实测代价

3. 32 个局部变量的上限
   - LDR/STR 的 6 位立即数偏移，反向决定了栈帧布局
   - 这个约束怎么一路传导到调用约定

写法：约束 → 权衡 → 结果。数字比形容词有用。
写完把 published 改成 true。
-->
