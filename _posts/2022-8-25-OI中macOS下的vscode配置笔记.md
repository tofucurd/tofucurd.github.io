---
layout:     post
title:      "OI中macOS下的vscode配置笔记"
subtitle:   "专为OI配置"
date:       2022-8-25
author:     "tofucurd"
header-img: "img/post-bg-1.jpg"
catalog: true
mathjax: true
tags: [macOS, 教程, vscode, clang]
---

#### 注：本文教程适用Apple Silicon和Intel的Mac

# 关于调试

自古以来，调试都有两种方法，一种是使用调试工具，一种是printf大法，本文两种方法都会有。

# step 1 安装command line tools

终端输入``xcode-select --install``安装command line tools

# step 2 添加万能头

终端输入``echo | clang++ -v -x c++ -E -``

进到``#include <...> search starts here:``下的第一个路径（我的是``/Library/Developer/CommandLineTools/SDKs/MacOSX.sdk/usr/include/c++/v1``），在目录下创建``bits`` 文件夹，再在里面添加文件``stdc++.h``：

```cpp
// C++ includes used for precompiling -*- C++ -*-

// Copyright (C) 2003-2022 Free Software Foundation, Inc.
//
// This file is part of the GNU ISO C++ Library.  This library is free
// software; you can redistribute it and/or modify it under the
// terms of the GNU General Public License as published by the
// Free Software Foundation; either version 3, or (at your option)
// any later version.

// This library is distributed in the hope that it will be useful,
// but WITHOUT ANY WARRANTY; without even the implied warranty of
// MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
// GNU General Public License for more details.

// Under Section 7 of GPL version 3, you are granted additional
// permissions described in the GCC Runtime Library Exception, version
// 3.1, as published by the Free Software Foundation.

// You should have received a copy of the GNU General Public License and
// a copy of the GCC Runtime Library Exception along with this program;
// see the files COPYING3 and COPYING.RUNTIME respectively.  If not, see
// <http://www.gnu.org/licenses/>.

/** @file stdc++.h
 *  This is an implementation file for a precompiled header.
 */

// 17.4.1.2 Headers

// C
#ifndef _GLIBCXX_NO_ASSERT
#include <cassert>
#endif
#include <cctype>
#include <cerrno>
#include <cfloat>
#include <ciso646>
#include <climits>
#include <clocale>
#include <cmath>
#include <csetjmp>
#include <csignal>
#include <cstdarg>
#include <cstddef>
#include <cstdio>
#include <cstdlib>
#include <cstring>
#include <ctime>
#include <cwchar>
#include <cwctype>

#if __cplusplus >= 201103L
#include <ccomplex>
#include <cfenv>
#include <cinttypes>
// #include <cstdalign>
#include <cstdbool>
#include <cstdint>
#include <ctgmath>
// #include <cuchar>
#endif

// C++
#include <algorithm>
#include <bitset>
#include <complex>
#include <deque>
#include <exception>
#include <fstream>
#include <functional>
#include <iomanip>
#include <ios>
#include <iosfwd>
#include <iostream>
#include <istream>
#include <iterator>
#include <limits>
#include <list>
#include <locale>
#include <map>
#include <memory>
#include <new>
#include <numeric>
#include <ostream>
#include <queue>
#include <set>
#include <sstream>
#include <stack>
#include <stdexcept>
#include <streambuf>
#include <string>
#include <typeinfo>
#include <utility>
#include <valarray>
#include <vector>

#if __cplusplus >= 201103L
#include <array>
#include <atomic>
#include <chrono>
#include <codecvt>
#include <condition_variable>
#include <forward_list>
#include <future>
#include <initializer_list>
#include <mutex>
#include <random>
#include <ratio>
#include <regex>
#include <scoped_allocator>
#include <system_error>
#include <thread>
#include <tuple>
#include <typeindex>
#include <type_traits>
#include <unordered_map>
#include <unordered_set>
#endif

#if __cplusplus >= 201402L
#include <shared_mutex>
#endif

#if __cplusplus >= 201703L
#include <any>
#include <charconv>
// #include <execution>
#include <filesystem>
#include <optional>
// #include <memory_resource>
#include <string_view>
#include <variant>
#endif

#if __cplusplus >= 202002L
#include <barrier>
#include <bit>
#include <compare>
#include <concepts>
#if __cpp_impl_coroutine
# include <coroutine>
#endif
#include <latch>
#include <numbers>
#include <ranges>
#include <span>
// #include <stop_token>
#include <semaphore>
// #include <source_location>
// #include <syncstream>
#include <version>
#endif

#if __cplusplus > 202002L
#include <spanstream>
#endif

```

保存即可。

# step 3 vscode配置

下面提供两种方法，两种方法均可用于Apple Silicon和Intel的Mac，**且两种方法互不干扰**。

# step 3.1 Code Runner+printf版

先安装必要插件：

- C/C++

- Code runner

## part 1 vscode基本设置

1. 在设置中找到``Files:Auto Save``，更改为``afterDelay``。

2. （可选）在设置中找到``Files:Default Language``，更改为``cpp``

3. （可选）在设置中找到``Workbench>Panel:Default Location``，更改为``right``，这个操作设置终端在屏幕的位置。

## part 2 C/C++ IntelliSense配置

打开你的工作目录（就是平时一个固定的写代码的地方），在vscode内置的资源管理器中在此目录下新建``.vscode``目录，再在该目录下新建文件``c_cpp_properties.json``：

```json
{
    "configurations": [
      {
        "name": "Mac",
        "includePath": [
            "${workspaceFolder}/**",
        ],
        "defines": [],
        "macFrameworkPath": [
        ],
        "compilerPath": "/usr/bin/clang",
        "cStandard": "c11",//可自行修改
        "cppStandard": "c++17",//可自行修改
        "intelliSenseMode": "clang-arm64"
      }
    ],
    "version": 4
}
```

再在``"includePath"``的中添加``echo | clang++ -v -x c++ -E -``中``#include <...> search starts here:``下面的一大坨路径（除了后面有个括号写的``framework directory``的那一个），注意要加引号和逗号：

```json
// 我的路径，仅供参考

"includePath": [
    "${workspaceFolder}/**",
    "/Library/Developer/CommandLineTools/SDKs/MacOSX.sdk/usr/include/c++/v1",
    "/Library/Developer/CommandLineTools/usr/lib/clang/13.1.6/include",
    "/Library/Developer/CommandLineTools/SDKs/MacOSX.sdk/usr/include",
    "/Library/Developer/CommandLineTools/usr/include"
],

```

再在``macFrameworkPath``中添加``echo | clang++ -v -x c++ -E -``中``#include <...> search starts here:``下面的路径中后面有个括号写的``framework directory``的那一个，同样要加引号：

```json
// 我的路径，仅供参考

"macFrameworkPath": [
    "/Library/Developer/CommandLineTools/SDKs/MacOSX.sdk/System/Library/Frameworks"
],

```

保存即可。

## part 3 Code Runner配置

进入Code Runner扩展设置，先把``Run In Terminal``打开，再打开``Executor Map``，在弹出的一大串命令中找到``"cpp": ``一行，替换为：

```json
//注意最后的逗号
"cpp": "cd $dir && clang++ -Wl,-stack_size,0x10000000 -std=c++17 -Wall -Ofast \"$fileName\" -o \"$fileNameWithoutExt\" && \"$fileNameWithoutExt\"",
```

保存即可。

## part 4 printf调试指南/技巧

写好一份代码后按快捷键``control+option+N``即可运行，注意必须要保存后的代码才有vscode的自动提示和自动补全。

### [使用Sanitizer](./2022-8-25-OI%E4%B8%ADSanitizer%E7%9A%84%E7%94%A8%E9%80%94)（强烈推荐）

# step 3.2 vscode内置调试版

先安装必要插件：

- C/C++

- CodeLLDB（为Mac提供本地调试兼容性支持）

## part 1

同step 3.1 part 1

## part 2

同step 3.2 part 2

## part 3 编译支持

在``c_cpp_properties.json``所在的``.vscode``目录下新建``task.json``，并将以下代码复制进去：

```json
{
    // See https://go.microsoft.com/fwlink/?LinkId=733558
    // for the documentation about the tasks.json format
    "version": "2.0.0",
    "tasks": [
        {
            "label": "Build with Clang",// 只是一个label，可以更改名字
            "type": "shell",
            "command": "clang++",
            "args": [//编译选项可以自行添加，与普通命令行选项没有区别
                "-g",//注意一定要加-g，否则无法调试
                "-std=c++17",
                "\"${fileDirname}/${fileBasenameNoExtension}.cpp\"",
                "-o",
                "\"${fileBasenameNoExtension}\""
            ],
            "group":{
                "kind": "build",
                "isDefault": true
            }
        }
    ]
}
```

如果想要更加深入了解，可以访问[官网](https://go.microsoft.com/fwlink/?LinkId=733558)。

## part 4 运行/调试支持

同样在``c_cpp_properties.json``所在的``.vscode``目录下新建``launch.json``，并将以下代码复制进去：

```json
{
    // 使用 IntelliSense 了解相关属性。 
    // 悬停以查看现有属性的描述。
    // 欲了解更多信息，请访问: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "type": "lldb",
            "request": "launch",
            "name": "Debug",
            "program": "${fileBasenameNoExtension}",//注意这个选项必须与task.json中编译出的位置相同（即-o后的那个路径）
            "args": [],
            "cwd": "${workspaceFolder}",
            "preLaunchTask": "Build with Clang"//注意这个选项必须与task.json中的label相同
        }
    ]
}
```

如果想要更加深入了解，可以访问[官网](https://go.microsoft.com/fwlink/?linkid=830387)。

## part 5 调试指南

点行号左边添加断电，按F5调试，可以在弹出的终端中输入。（剩下的相信大家都会了吧）

# 完结撒花！