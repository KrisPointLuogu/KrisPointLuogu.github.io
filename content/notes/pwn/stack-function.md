---
title: Stack 和 C 语言函数
date: 2026-11-16
tags:
  - pwn
  - stack
---
stack 这个先进后出的性质，天生就适合做嵌套的记录工具，一层层一层层的。

解析一下 64 位 `stack_push.s` 的代码，看 f8 的代码：

```asm
push rbp       ; rsp -= 8
mov rbp, rsp   ; rbp = rsp

; 一段 code

pop rbp        ; rsp += 8, rbp = 函数开头的 rbp
ret            ; pop rip, rsp += 8
```

分析一下程序的栈：

```text
[main 栈] -> [参数] -> [main call f8 时的 rip (ret 靠它回溯)] -> [rbp] -> [函数栈]
high ----------------------------------------------------------------> low
```

于是，一位天才提出了：可不可以从低地址往上写，写多了就溢出到返回地址，就可以随意控制程序走向了？

还真能。

这就是 Stack Overflow。

~~好了，你现在已经会栈溢出了，现在来构建 ROP 链，做 SROP BROP 吧()~~

## stack_push.c

```c
// 一个参数
void f1(int a)
{
    a = a + 1;
}
// 两个参数
void f2(int a, int b)
{
    a = a + 1;
    b = b + 1;
}
// 四个参数
void f4(int a, int b, int c, int d)
{
    a = a + 1;
    b = b + 1;
    c = c + 1;
    d = d + 1;
}
// 六个参数
void f6(int a, int b, int c, int d, int e, int f)
{
    a = a + 1;
    b = b + 1;
    c = c + 1;
    d = d + 1;
    e = e + 1;
    f = f + 1;
}
// 八个参数
void f8(int a, int b, int c, int d, int e, int f, int g, int h)
{
    a = a + 1;
    b = b + 1;
    c = c + 1;
    d = d + 1;
    e = e + 1;
    f = f + 1;
    g = g + 1;
    h = h + 1;
}
int main(int argc, char *argv[])
{
    f1(2);
    f2(1, 2);
    f4(1, 2, 3, 4);
    f6(1, 2, 3, 4, 5, 6);
    f8(1, 2, 3, 4, 5, 6, 7, 8);
    /*
        [BITS 32]
        函数用栈传递参数，eax做返回值

        [BITS 64]
        在 0-6 个参数时候，依次采用 rdi, rsi, rdx, rcx, r8, r9传递参数
        rax返回
    */
    return 0;
}
```

## stack_push.s

```asm
	.file	"stack_push.c"
	.intel_syntax noprefix
	.text
	.globl	f1
	.type	f1, @function
f1:
.LFB0:
	.cfi_startproc
	endbr64
	push	rbp
	.cfi_def_cfa_offset 16
	.cfi_offset 6, -16
	mov	rbp, rsp
	.cfi_def_cfa_register 6
	mov	DWORD PTR -4[rbp], edi
	add	DWORD PTR -4[rbp], 1
	nop
	pop	rbp
	.cfi_def_cfa 7, 8
	ret
	.cfi_endproc
.LFE0:
	.size	f1, .-f1
	.globl	f2
	.type	f2, @function
f2:
.LFB1:
	.cfi_startproc
	endbr64
	push	rbp
	.cfi_def_cfa_offset 16
	.cfi_offset 6, -16
	mov	rbp, rsp
	.cfi_def_cfa_register 6
	mov	DWORD PTR -4[rbp], edi
	mov	DWORD PTR -8[rbp], esi
	add	DWORD PTR -4[rbp], 1
	add	DWORD PTR -8[rbp], 1
	nop
	pop	rbp
	.cfi_def_cfa 7, 8
	ret
	.cfi_endproc
.LFE1:
	.size	f2, .-f2
	.globl	f4
	.type	f4, @function
f4:
.LFB2:
	.cfi_startproc
	endbr64
	push	rbp
	.cfi_def_cfa_offset 16
	.cfi_offset 6, -16
	mov	rbp, rsp
	.cfi_def_cfa_register 6
	mov	DWORD PTR -4[rbp], edi
	mov	DWORD PTR -8[rbp], esi
	mov	DWORD PTR -12[rbp], edx
	mov	DWORD PTR -16[rbp], ecx
	add	DWORD PTR -4[rbp], 1
	add	DWORD PTR -8[rbp], 1
	add	DWORD PTR -12[rbp], 1
	add	DWORD PTR -16[rbp], 1
	nop
	pop	rbp
	.cfi_def_cfa 7, 8
	ret
	.cfi_endproc
.LFE2:
	.size	f4, .-f4
	.globl	f6
	.type	f6, @function
f6:
.LFB3:
	.cfi_startproc
	endbr64
	push	rbp
	.cfi_def_cfa_offset 16
	.cfi_offset 6, -16
	mov	rbp, rsp
	.cfi_def_cfa_register 6
	mov	DWORD PTR -4[rbp], edi
	mov	DWORD PTR -8[rbp], esi
	mov	DWORD PTR -12[rbp], edx
	mov	DWORD PTR -16[rbp], ecx
	mov	DWORD PTR -20[rbp], r8d
	mov	DWORD PTR -24[rbp], r9d
	add	DWORD PTR -4[rbp], 1
	add	DWORD PTR -8[rbp], 1
	add	DWORD PTR -12[rbp], 1
	add	DWORD PTR -16[rbp], 1
	add	DWORD PTR -20[rbp], 1
	add	DWORD PTR -24[rbp], 1
	nop
	pop	rbp
	.cfi_def_cfa 7, 8
	ret
	.cfi_endproc
.LFE3:
	.size	f6, .-f6
	.globl	f8
	.type	f8, @function
f8:
.LFB4:
	.cfi_startproc
	endbr64
	push	rbp
	.cfi_def_cfa_offset 16
	.cfi_offset 6, -16
	mov	rbp, rsp
	.cfi_def_cfa_register 6
	mov	DWORD PTR -4[rbp], edi
	mov	DWORD PTR -8[rbp], esi
	mov	DWORD PTR -12[rbp], edx
	mov	DWORD PTR -16[rbp], ecx
	mov	DWORD PTR -20[rbp], r8d
	mov	DWORD PTR -24[rbp], r9d
	add	DWORD PTR -4[rbp], 1
	add	DWORD PTR -8[rbp], 1
	add	DWORD PTR -12[rbp], 1
	add	DWORD PTR -16[rbp], 1
	add	DWORD PTR -20[rbp], 1
	add	DWORD PTR -24[rbp], 1
	add	DWORD PTR 16[rbp], 1
	add	DWORD PTR 24[rbp], 1
	nop
	pop	rbp
	.cfi_def_cfa 7, 8
	ret
	.cfi_endproc
.LFE4:
	.size	f8, .-f8
	.globl	main
	.type	main, @function
main:
.LFB5:
	.cfi_startproc
	endbr64
	push	rbp
	.cfi_def_cfa_offset 16
	.cfi_offset 6, -16
	mov	rbp, rsp
	.cfi_def_cfa_register 6
	sub	rsp, 16
	mov	DWORD PTR -4[rbp], edi
	mov	QWORD PTR -16[rbp], rsi
	mov	edi, 2
	call	f1
	mov	esi, 2
	mov	edi, 1
	call	f2
	mov	ecx, 4
	mov	edx, 3
	mov	esi, 2
	mov	edi, 1
	call	f4
	mov	r9d, 6
	mov	r8d, 5
	mov	ecx, 4
	mov	edx, 3
	mov	esi, 2
	mov	edi, 1
	call	f6
	push	8
	push	7
	mov	r9d, 6
	mov	r8d, 5
	mov	ecx, 4
	mov	edx, 3
	mov	esi, 2
	mov	edi, 1
	call	f8
	add	rsp, 16
	mov	eax, 0
	leave
	.cfi_def_cfa 7, 8
	ret
	.cfi_endproc
.LFE5:
	.size	main, .-main
	.ident	"GCC: (Ubuntu 15.3.0-4ubuntu1) 15.3.0"
	.section	.note.GNU-stack,"",@progbits
	.section	.note.gnu.property,"a"
	.align 8
	.long	1f - 0f
	.long	4f - 1f
	.long	5
0:
	.string	"GNU"
1:
	.align 8
	.long	0xc0000002
	.long	3f - 2f
2:
	.long	0x3
3:
	.align 8
4:
```
