---
layout: post
title: "What Sanitizers Are Actually For"
subtitle: "Set up for competitive programming"
date: 2022-8-25
lang: en
ref: sanitizer
permalink: /en/2022/08/25/sanitizers/
tags: [Windows, macOS, Linux, gcc, clang]
---

# Availability

- Windows: only clang has them
- Linux: both gcc and clang have them
- macOS: only Apple clang has them

# Usage

A sanitizer is a tool built into gcc/clang that helps you find memory leaks, out-of-bounds and negative indices, integer overflow, and the rest of the family of things that cause runtime errors. There are two relevant ones: AddressSanitizer and UndefinedBehaviorSanitizer. Just add `-g -fsanitize=address,undefined` when you invoke clang. For example:

```cpp
#include<bits/stdc++.h>
using namespace std;
int main(){
    int a[5];
    for(int i=0;i<5;i++)cin>>a[i];
    for(int i=-1;i<=5;i++)printf("%d ",a[i]);
    a[1]=(a[1]+100)*INT_MAX;
    return 0;
}
```

With the input `1 2 3 4 5`, a build without sanitizers prints:

```
67812992 1 2 3 4 5 -2068840411
```

Garbage, and not a word of warning.

With sanitizers, it prints:

```
test.cpp:6:40: runtime error: index -1 out of bounds for type 'int [5]'
SUMMARY: UndefinedBehaviorSanitizer: undefined-behavior test.cpp:6:40 in
test.cpp:6:40: runtime error: load of address 0x00016dc3319c with insufficient space for an object of type 'int'
=================================================================
==73013==ERROR: AddressSanitizer: stack-buffer-underflow on address 0x00016dc3319c at pc 0x0001021cfb2c bp 0x00016dc33170 sp 0x00016dc33168
READ of size 4 at 0x00016dc3319c thread T0
    #0 0x1021cfb28 in main test.cpp:6
    #1 0x102249088 in start+0x204 (dyld:arm64e+0x5088)

Address 0x00016dc3319c is located in stack of thread T0 at offset 28 in frame
    #0 0x1021cf664 in main test.cpp:3

  This frame has 1 object(s):
    [32, 52) 'a' (line 4) <== Memory access at offset 28 underflows this variable
SUMMARY: AddressSanitizer: stack-buffer-underflow test.cpp:6 in main
[...]
==73013==ABORTING
```

The first four lines already point at both bugs, at line 6 column 40. The pile of addresses that follows can be ignored — you only need the location and the message to pin the bug down exactly. It is a fantastic tool.
