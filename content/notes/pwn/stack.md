---
title: 栈
date: 2026-11-16
tags:
  - pwn
  - amd64
---
## 问题：什么是栈

栈是一种神奇的东西，它先进后出，后进先出，在内存中从上往下布局，很适合用来寄存状态。

In fact，在 C 语言里面就是用 sp 寄存器开辟函数内部临时变量的。

## 问题：栈的操作

```asm
; [BITS 32]
push reg   ; sub esp, 4; mov [esp], reg
pop reg    ; mov reg, [esp]; add esp, 4
```

```asm
; [BITS 64]
push reg   ; sub rsp, 8; mov [rsp], reg
pop reg    ; mov reg, [rsp]; add rsp, 8
```
