---
layout:     post
title:      "OI中Sanitizer的用途"
subtitle:   "专为OI配置"
date:       2022-8-25
author:     "tofucurd"
header-img: "img/post-bg-1.jpg"
catalog: true
mathjax: true
tags: [Windows, macOS, Linux, gcc, clang]
---

# 环境要求

- Windows下，只有clang有

- Linux下，gcc/clang都有

- macOS下，只有Apple clang有

# 使用说明

Sanitizer是一种工具，内置于gcc/clang中，可以帮助查找内存泄漏，死循环，负下标/超下标，爆int等一系列可能导致RE的工具，具体而言有两个：AddressSanitizer和UndefinedBehaviorSanitizer。使用时直接在调用clang编译时添加``-g -fsanitize=address,undefined``即可。例如下面的一份代码：

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

当输入``1 2 3 4 5``时，如果没有使用Sanitizer，会输出：

```
67812992 1 2 3 4 5 -2068840411 
```

可见输出了乱码且没有任何提示。

而当使用Sanitizer后，会输出：

```
test.cpp:6:40: runtime error: index -1 out of bounds for type 'int [5]'
SUMMARY: UndefinedBehaviorSanitizer: undefined-behavior test.cpp:6:40 in 
test.cpp:6:40: runtime error: load of address 0x00016dc3319c with insufficient space for an object of type 'int'
0x00016dc3319c: note: pointer points here
  98 5c 27 02 01 80 5c 40  01 00 00 00 02 00 00 00  03 00 00 00 04 00 00 00  05 00 00 00 01 00 00 00
              ^ 
SUMMARY: UndefinedBehaviorSanitizer: undefined-behavior test.cpp:6:40 in 
=================================================================
==73013==ERROR: AddressSanitizer: stack-buffer-underflow on address 0x00016dc3319c at pc 0x0001021cfb2c bp 0x00016dc33170 sp 0x00016dc33168
READ of size 4 at 0x00016dc3319c thread T0
    #0 0x1021cfb28 in main test.cpp:6
    #1 0x102249088 in start+0x204 (dyld:arm64e+0x5088)

Address 0x00016dc3319c is located in stack of thread T0 at offset 28 in frame
    #0 0x1021cf664 in main test.cpp:3

  This frame has 1 object(s):
    [32, 52) 'a' (line 4) <== Memory access at offset 28 underflows this variable
HINT: this may be a false positive if your program uses some custom stack unwind mechanism, swapcontext or vfork
      (longjmp and C++ exceptions *are* supported)
SUMMARY: AddressSanitizer: stack-buffer-underflow test.cpp:6 in main
Shadow bytes around the buggy address:
  0x00702dba65e0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x00702dba65f0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x00702dba6600: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x00702dba6610: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x00702dba6620: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
=>0x00702dba6630: f1 f1 f1[f1]00 00 04 f3 f3 f3 f3 f3 00 00 00 00
  0x00702dba6640: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x00702dba6650: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x00702dba6660: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x00702dba6670: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x00702dba6680: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
Shadow byte legend (one shadow byte represents 8 application bytes):
  Addressable:           00
  Partially addressable: 01 02 03 04 05 06 07 
  Heap left redzone:       fa
  Freed heap region:       fd
  Stack left redzone:      f1
  Stack mid redzone:       f2
  Stack right redzone:     f3
  Stack after return:      f5
  Stack use after scope:   f8
  Global redzone:          f9
  Global init order:       f6
  Poisoned by user:        f7
  Container overflow:      fc
  Array cookie:            ac
  Intra object redzone:    bb
  ASan internal:           fe
  Left alloca redzone:     ca
  Right alloca redzone:    cb
==73013==ABORTING
[1]    73013 abort      "test"
```

首先开头四行便指出了2处在6行40列的错误，后面的一大坨地址可以不用管，你只需要找到错误的位置和报错信息即可准确定位，堪称神器。