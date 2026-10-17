---
layout: post
title: "Setting Up VS Code for Competitive Programming on macOS"
subtitle: "Tuned for OI"
date: 2022-8-25
lang: en
ref: vscode-macos
permalink: /en/2022/08/25/vscode-macos/
tags: [macOS, tutorial, vscode, clang]
---

#### Note: this guide works on both Apple Silicon and Intel Macs.

# On debugging

There have always been two ways to debug: use a debugger, or use printf. This post covers both.

# step 1 Install the command line tools

Run `xcode-select --install` in a terminal.

# step 2 Add the universal header

Run `echo | clang++ -v -x c++ -E -` and look at the first path under `#include <...> search starts here:` (mine is `/Library/Developer/CommandLineTools/SDKs/MacOSX.sdk/usr/include/c++/v1`). Create a `bits` folder there and put a file named `stdc++.h` inside it.

That file is simply GCC's universal header — copy it from any GCC installation or from the GCC source tree. (It is omitted here; nothing about it is macOS-specific, and retyping 150 lines of `#include` helps nobody.)

Save.

# step 3 Configure VS Code

Two approaches follow. Both work on Apple Silicon and Intel, and **they do not interfere with each other**.

# step 3.1 Code Runner + printf

Install the extensions:

- C/C++
- Code Runner

## part 1 Basic settings

1. Find `Files: Auto Save` and set it to `afterDelay`.

2. (optional) Find `Files: Default Language` and set it to `cpp`.

3. (optional) Find `Workbench > Panel: Default Location` and set it to `right` — this decides where the terminal sits on screen.

## part 2 C/C++ IntelliSense

Open your working directory (the fixed place where you keep code), create a `.vscode` folder in it, and inside that a file `c_cpp_properties.json`:

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
        "cStandard": "c11",
        "cppStandard": "c++17",
        "intelliSenseMode": "clang-arm64"
      }
    ],
    "version": 4
}
```

Then add to `"includePath"` all the paths listed under `#include <...> search starts here:` in the `echo | clang++ -v -x c++ -E -` output — except the one followed by a parenthesised `framework directory`. Quote them and mind the commas:

```json
// mine, for reference only

"includePath": [
    "${workspaceFolder}/**",
    "/Library/Developer/CommandLineTools/SDKs/MacOSX.sdk/usr/include/c++/v1",
    "/Library/Developer/CommandLineTools/usr/lib/clang/13.1.6/include",
    "/Library/Developer/CommandLineTools/SDKs/MacOSX.sdk/usr/include",
    "/Library/Developer/CommandLineTools/usr/include"
],

```

Then put the `framework directory` path into `macFrameworkPath`, again quoted:

```json
// mine, for reference only

"macFrameworkPath": [
    "/Library/Developer/CommandLineTools/SDKs/MacOSX.sdk/System/Library/Frameworks"
],

```

Save.

## part 3 Code Runner

Open the Code Runner extension settings, turn on `Run In Terminal`, then open `Executor Map`, find the `"cpp": ` line among the long list of commands, and replace it with:

```json
// note the trailing comma
"cpp": "cd $dir && clang++ -Wl,-stack_size,0x10000000 -std=c++17 -Wall -Ofast \"$fileName\" -o \"$fileNameWithoutExt\" && \"./$fileNameWithoutExt\"",
```

Save.

## part 4 printf-style debugging

With code written, press `control+option+N` to run it. Note that VS Code's completion and suggestions only work on a saved file.

### [Using sanitizers](https://tofucurd.github.io/2022/08/25/OI中Sanitizer的用途/) (strongly recommended)

# step 3.2 VS Code's built-in debugger

Install the extensions:

- C/C++
- CodeLLDB (local debugging support for macOS)

## part 1

Same as step 3.1 part 1.

## part 2

Same as step 3.1 part 2.

## part 3 Build support

In the same `.vscode` folder as `c_cpp_properties.json`, create `tasks.json` and paste this in:

```json
{
    "version": "2.0.0",
    "tasks": [
        {
            "label": "Build with Clang", // just a label, rename it if you like
            "type": "shell",
            "command": "clang++",
            "args": [ // compiler flags go here, no different from the command line
                "-g", // -g is required, otherwise debugging will not work
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

For more detail, see the [documentation](https://go.microsoft.com/fwlink/?LinkId=733558).

## part 4 Run and debug support

Also in the `.vscode` folder, create `launch.json` and paste this in:

```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "type": "lldb",
            "request": "launch",
            "name": "Debug",
            "program": "${fileBasenameNoExtension}", // must match the output path in task.json, i.e. what follows -o
            "args": [],
            "cwd": "${workspaceFolder}",
            "preLaunchTask": "Build with Clang" // must match the label in task.json
        }
    ]
}
```

For more detail, see the [documentation](https://go.microsoft.com/fwlink/?linkid=830387).

## part 5 Debugging

Click to the left of a line number to set a breakpoint, press F5, and type into the terminal that appears. (The rest you can work out.)

# Done!
