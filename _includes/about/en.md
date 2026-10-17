I am an undergraduate in Electrical and Computer Engineering at the
[ZJU-UIUC Institute](https://zjui.intl.zju.edu.cn/), currently in Haining, China.
I will be on the Urbana-Champaign campus from Fall 2027.

My interests are compilers and low-level systems — I like problems where the output
has to actually run, and where "it works" is something you can measure.

## What I have built

**[LLVMIRToLC3Pass](https://github.com/tofucurd/LLVMIRToLC3Pass)** — an out-of-tree
LLVM pass, written in C++, that lowers LLVM IR generated from C down to LC-3
assembly: a 16-bit ISA with eight registers. It covers `phi`, `select`, `switch`,
`getelementptr`, variadic calls, and a small libc (`printf`, `scanf`, `sprintf`,
`sscanf`). It is tested by 35 cases in three tiers; 16 of them assemble the generated
code and run it on an LC-3 simulator, comparing stdout against expected output.

## Before that

Competitive programming (Chinese OI), 2019–2023. The habit I kept from it is
measuring constant factors instead of trusting asymptotics.

I write about what I learn in [my blog](/archive/), mostly in Chinese.

## Contact

[liu_yi_le@icloud.com](mailto:liu_yi_le@icloud.com) · [GitHub](https://github.com/tofucurd)
