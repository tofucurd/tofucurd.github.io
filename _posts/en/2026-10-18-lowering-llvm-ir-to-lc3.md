---
layout: post
title: "From LLVM IR to LC-3 Assembly"
subtitle: "Notes on the trade-offs in one pass"
date: 2026-10-18
lang: en
ref: lc3-design
permalink: /en/2026/10/18/lowering-llvm-ir-to-lc3/
mathjax: true
tags: [LLVM, compilers, LC-3]
published: true
---

Project: <https://github.com/tofucurd/LLVMIRToLC3Pass>

Something I wrote in my first semester: a compiler from C to LC-3, a 16-bit teaching ISA with eight registers whose ALU does only AND, ADD and NOT. It started as the assembly assignments for ECE 220; afterwards I used it to do all the assembly MPs.

## Why a pass, not a backend

The LLVM backend framework is fairly involved. TableGen takes some of the weight off, but the bar is still high. And an educational architecture like LC-3 is severely constrained — it can never really be full C compatible: it is purely 16-bit and word-addressed rather than byte-addressed, it has no arithmetic wider than 16 bits, an ALU that only does AND, ADD and NOT, load/store register offsets limited to [-16, 15], and branches that reach at most 1024 addresses. LC-3 was never going to carry a program of any real size, so translating LLVM IR directly was the most reasonable and most convenient option.

## The LC-3 calling convention

For uniformity and ease of understanding, LC-3 passes arguments and return values purely on the stack: R5 is fp, R6 is sp, the stack grows from high addresses (top at 0xFE00) downward, R7 holds the return address, and all of them are callee-saved. Since LC-3 is 16-bit only, there is no big-endian / little-endian distinction. The stack frame looks like this (Watt is the caller, Volt the callee):

![fig.1](https://tofucurd.github.io/img/post-2026-10-18-1.png "Markdown")

## Overall approach

LC-3 cannot hold a large program, and its ABI covers only string I/O, so there is usually a single LLVM IR module. `LLVMIRToLC3Pass::run()` therefore takes the module and translates block by block, drawing no distinction between pointers and variables: every virtual register lives in a stack slot as a local variable. Because this is only a pass, the output has to be written by hand through `ToolOutputFile` and `raw_string_ostream`. The flow is: preprocess the IR so that it is easier to translate instruction by instruction, then translate it instruction by instruction, with extra handling for a few special cases.

## Implementation notes

### 1. PHI nodes — the hard part

PHI nodes are LLVM IR's mechanism for merging control flow, and they always sit at the top of a block. Together with blocks, PHI nodes handle complex branching efficiently. The difficulty is that a PHI has to know which block it was entered from. One trick that works is to reuse the R7 register: at the end of every block, just before branching, LC-3's `LEA` loads the address of the current block's LABEL into R7, so the PHI can immediately tell where the jump came from. The two uses of R7 never overlap in lifetime, so there is no conflict.

Saving the information in R7 at branch time:

```cpp
if (auto *BranchI = dyn_cast<BranchInst>(&I)) {
          FuncInstBufferStream << "\tLEA\t\tR7, " << BBName << "\n";
/*...*/
```

The PHI node itself:

```cpp
int ResOff = -getIndex(&I, ValueOffsetMap, ValueOffsetCounter);

FuncInstBufferStream << "\tNOT\t\tR0, R7\n"
                     << "\tADD\t\tR0, R0, #1\n";
int ArgSize = PHIN->getNumIncomingValues();
int EndLableID = TempLabelCounter + ArgSize;
for (unsigned int i = 0; i < ArgSize; ++i) {

BasicBlock *SrcBB = PHIN->getIncomingBlock(i);
std::string SrcBBName = getIndex(SrcBB, BBNameMap, BBNameCounter);

++TempLabelCounter;
if (i < ArgSize - 1) {
   FuncInstBufferStream << "\tLEA\t\tR1, " << SrcBBName << "\n"
                        << "\tADD\t\tR1, R1, R0\n"
                        << "\tBRnp\tPHI_NEXT_" << TempLabelCounter
                        << "\n";
}

Value *Val = PHIN->getIncomingValue(i);
if (int ValID = addImmidiate(Val, ImmBufferStream, ImmFlag,
                              ImmIDMap, ImmIDCounter)) {
   FuncInstBufferStream << "\tLD\t\tR1, VALUE_" << ValID << "\n";
} else {
   int ValOff = -getIndex(Val, ValueOffsetMap, ValueOffsetCounter);
   FuncInstBufferStream << "\tLDR\t\tR1, R5, #" << ValOff << "\n";
}

FuncInstBufferStream << "\tSTR\t\tR1, R5, #" << ResOff << "\n";
if (i < ArgSize - 1) {
   FuncInstBufferStream << "\tBR\t\tPHI_NEXT_" << EndLableID << "\n";
}
FuncInstBufferStream << "PHI_NEXT_" << TempLabelCounter << "\n";
}
```

### 2. Index management

Every object that has to be referred to repeatedly — blocks, variables — gets a dictionary index built from a DenseMap and a counter; the core is `getIndex`. For a block it returns an id, which conveniently becomes the LABEL in the LC-3 assembly. For a variable the returned index is exactly its offset relative to R5 — positive for locals, negative for arguments. For immediates, LC-3's imm field is too small, so every immediate is stored in the space after the BR at the end of its block and addressed through the LABEL `VALUE_<Index>`. The implementation:

```cpp
int getIndex(Value *Val, DenseMap<Value *, int> &Map, int &Counter) {
  if (Map.count(Val) == 0) {
    if (auto *Arg = dyn_cast<Argument>(Val)) {
      unsigned Index = Arg->getArgNo();
      Map[Val] = -4 - Index;
    } else {
      Map[Val] = Counter++;
    }
  }
  return Map[Val];
}
std::string getIndex(BasicBlock *BB, DenseMap<Value *, std::string> &Map,
                     int &Counter) {
  if (Map.count(BB) == 0) {
    Map[BB] = BB->getParent()->getName().str() + "_" + getBBName(BB) + "_" +
              std::to_string(Counter++);
  }
  return Map[BB];
}
```

### 3. Preprocessing

To make the later translation easier, the IR is preprocessed first. The main items:

- LLVM IR has a class of `Intrinsic` calls; the most common and the one that must be handled is min/max, which LC-3 has nothing for, so these are turned into `CmpInst` and handled in the next stage
- LC-3 has no subtract instruction. Writing negate-and-add every time a subtraction is needed would be too expensive, so for the two most frequent patterns, `icmp X, C` and `sub X, C`, the constant is precomputed into `icmp X, -C` and `add X, -C`. Note that the `icmp` is no longer used in its original sense afterwards — it has to be matched by translation logic written for it specifically. This is a preprocessing step done for convenience, in the spirit of `constexpr`
- Generalize special instructions: `br (icmp eq/ne X, C)` becomes `switch`; `disjoint or` becomes `add`
- Pass some instructions through, e.g. `trunc`
- Shifting right by a constant is precomputed into `udiv X, (1 << Constant)`

### 4. Translating binary operations

LC-3 assembly has only AND, ADD and NOT, so the other LLVM IR operations have to be synthesized in software:

- `mul`: repeated addition, assuming non-negative operands to save a few instructions
- `shl`: repeated doubling
- `udiv` / `urem`: repeated subtraction
- `lshr`: two pointers walk the source and the destination one bit at a time, setting the offset from the real bit width so that writes to the destination are delayed
- a `-signed-mul` option checks the sign first

### 5. I/O API

To make I/O possible from C, a small set of I/O APIs is built in: at each CallInst the callee name is checked and the code inlined. See `LC3.h`.

### 6. Function calls

Function calls follow the calling convention from ECE 220 exactly, jumping with JSR, so the offset is limited to ±1024 words. See the source for the implementation.

## Performance against mp11's tiny compiler

The numbers below are **dynamically executed** machine instructions, from entry to `main` through its return: stack-frame traffic, `RET`, and any library code that actually runs are included; startup and the printing after return are not. Every successful case was executed instruction by instruction on an LC-3 simulator, with the PC and machine-code trace replayed and the return value checked. mp11 has no optimization switch; the pass is fed IR from `clang -O0`, so that clang's optimizations are not counted as lowering quality.

| Test | mp11 | this pass | reduction |
| :-- | ---: | ---: | ---: |
| Modular exponentiation: 3¹³⁷ mod 1009 | 19,456 | 11,312 | 41.9% |
| Three nested loops | 34,813 | 26,957 | 22.6% |
| Integer square roots, 32 at a time | 51,172 | 35,668 | 30.3% |
| Boolean branching and scalar recurrence | 48,854 | 26,316 | 46.1% |
| **total** | **154,295** | **100,253** | **35.0%** |

## Limitations of this pass

- At most 32 frame slots. `LDR`/`STR` take a 6-bit signed immediate offset (-16..15) and the frame pointer is fixed at R5, so a function can have at most 32 slots.
- Calls are range-limited: `JSR` uses an 11-bit offset, i.e. ±1024 words.
- No floating point; unsigned values are treated as signed.
- A number of instructions are still not implemented.

## A note on AI

The last two commits on the `main` branch were used as material for testing Codex. The hand-written version lives on the `handwritten` branch, and all of the ECE 220 assembly MPs were done with that one.
