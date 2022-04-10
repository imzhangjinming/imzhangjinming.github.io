---
title: 深入理解计算机系统 第三章 程序的机器级表示
date: 2021-11-20 00:52:15
categories:
- notes
tags:
---
第三章 程序的机器级表示

<!--more-->

# 第三章 程序的机器级表示

## 3.1 从历史角度看

  Intel 的处理器产品线，通常又被叫做 x86 处理器，已经经历了一个很长时间的发展和进步

原书这里介绍了 Intel 历史上的经典处理器型号，我把它省略了吧，因为很像 Intel 的发家史

AMD 的全称是 Advanced Micro Devices

obsolete adj. 废弃的；老式的



## 3.2 程序的编码

假设我们编写了两个C文件 p1.c 和 p2.c

现在用下面的命令行进行编译

```shell
linux> gcc -Og -o p p1.c p2.c
```

`-Og` 要求编译器进行能够保留原始C代码结构的优化，不要过度优化

执行这条命令后，gcc 编译器会调用一系列程序将 **源代码** 变成 **可执行文件**

首先，**预处理器**（**preprocessor**）会处理所有 预编译代码 和 宏

然后**编译器**（**compiler**）将C语言翻译成对应的 汇编语言 版本

接着，**汇编器**（**assembler**）将 汇编语言 翻译成 全是二进制机器语言 的目标文件

最后，**链接器**（**linker**）出马，将之前的 目标文件 与 库代码等连接到一起，生成最后的可执行文件



### 3.2.1 机器级代码

机器级代码的格式和行为全部由 **指令集架构**（instruction set architecture,ISA）决定

elaborate adj. 复杂的

机器级的代码使用的是**虚拟地址**，操作系统负责 虚拟地址 和 真实的物理地址 之间的转换



### 3.2.2 一个代码例子

```c
long mult2(long, long); 

void multstore(long x, long y, long *dest) { 
    long t = mult2(x, y);
    *dest = t; 
}
```

我们把文件命名为 mstore.c 并执行以下命令

```shell
linux> gcc -Og -S mstore.c
```

`-S` 选项要求 gcc 生成 mstore.s 后停止，不再继续后面的工作

如果执行

```shell
linux> gcc -Og -c mstore.c
```

`-c` 选项会使 gcc 在生成 mstore.o 后停止

 

如果想查看目标文件里的内容（.o 文件），可以使用 objdump 反汇编工具（disassemblers）

```shell
linux> objdump -d mstore.o
```

在虚拟机中执行完上面这行命令后的输出是

```shell
mstore.o：     文件格式 elf64-x86-64


Disassembly of section .text:

0000000000000000 <multstore>:
   0:	53                   	push   %rbx
   1:	48 89 d3             	mov    %rdx,%rbx
   4:	e8 00 00 00 00       	callq  9 <multstore+0x9>
   9:	48 89 03             	mov    %rax,(%rbx)
   c:	5b                   	pop    %rbx
   d:	c3                   	retq 
```



现在再编写一个文件 main.c 来生成真正的可执行文件

```c
#include <stdio.h> 

void multstore(long, long, long *); 

int main() { 
    long d; 
    multstore(2, 3, &d); 
    printf("2 * 3 --> %ld\n", d); 
    return 0; 
} 

long mult2(long a, long b) { 
    long s = a * b; 
    return s; 
}
```

执行下面的命令生成可执行文件

```shell
linux> gcc -Og -o prog main.c mstore.c
```



## 3.3 数据格式

在 x86 系列发展的过程中，先是把 16-bit 数据称为 word 

随着该系列处理器的继续发展，把 32-bit 数据称为 double words ，64-bit 数据称为 quad words

下表列出了与C语言中一些类型对应的 Intel 数据类型

{% asset_img Snipaste_2021-09-29_09-18-51.png %}



## 3.4 访问信息

一个 x86-64 CPU 拥有 16 个 64 位**通用寄存器**（general purpose register）

{% asset_img 208.png %}

### 3.4.1 操作数指定符 operand specifiers

操作数分为三类：直接数、寄存器 和 内存地址



有很多寻址模式 addressing modes ，它们的通式可以表示为 $Imm(r_b,r_i,s)$ ，它表示的地址是 $Imm+R[r_b]+R[r_i*s]$

{% asset_img Snipaste_2021-10-21_20-52-08.png %}

### Practice Problem 3.1

Assume the following values are stored at the indicated memory addresses and registers:

| address | value | register | value |
| :-------: | :-----: | :--------: | :-----: |
| 0x100 | 0xFF | %rax | 0x100 |
| 0x104 | 0xAB | %rcx | 0x1 |
| 0x108 | 0x13 | %rdx | 0x3 |
| 0x10c | 0x11 | | |

Fill in the following table showing the values for the indicated operands:

| operand        | value |
| -------------- | ----- |
| %rax           | 0x100 |
| 0x104          | 0xAB  |
| $0x108         | 0x108 |
| (%rax)         | 0xFF  |
| 4(%rax)        | 0xAB  |
| 9(%rax,%rdx)   | 0x11  |
| 260(%rcx,%rdx) | 0x13  |
| 0xFC(,%rcx,4)  | 0xFF  |
| (%rax,%rdx,4)  | 0x11  |



### 3.4.2 移动数据的指令

`MOV` 类指令

{% asset_img Snipaste_2021-10-20_21-15-29.png %}

通常，`MOV` 类指令只会改变目标寄存器的**指定位**。唯一的**例外**是：当指令 `movl` 的目标操作数是一个寄存器时，该指令会把目标寄存器的高32位（高4字节）**置零**，这与 x86-64 架构有关



```assembly
movl $0x4050,%eax 		Immediate--Register, 4 bytes
movw %bp,%sp 			Register--Register, 2 bytes
movb (%rdi,%rcx),%al 	Memory--Register, 1 byte
movb $-17,(%esp) 		Immediate--Memory, 1 byte
movq %rax,-12(%rbp) 	Register--Memory, 8 bytes
```



有两组专门将小数据拷贝到大寄存器里的指令

`movz` 类：目标寄存器的高位用 0 填充（zero）

{% asset_img Snipaste_2021-10-21_08-36-39.png %}

`movs` 类：目标寄存器的高位用 符号位 填充（sign）

{% asset_img Snipaste_2021-10-21_08-36-57.png %}



### Practice Problem 3.2

For each of the following lines of assembly language, determine the appropriate instruction suffix based on the operands. (For example, `mov` can be rewritten as `movb`, `movw`, `movl`, or `movq`.)

```assembly
movl %eax, (%rsp)
movw (%rax), %dx
movb $0xFF, %bl
movb (%rsp,%rdx,4), %dl
movq (%rdx), %rax
movw %dx, (%rax)
```



### Practice Problem 3.3

Each of the following lines of code generates an error message when we invoke the assembler. Explain what is wrong with each line.

```assembly
movb $0xF, (%ebx)			ebx （32位）不能用作地址寄存器
movl %rax, (%rsp)			应该使用 movq 指令
movw (%rax),4(%rsp)			不能从一个内存地址直接向另一个内存地址拷贝数据
movb %al,%sl				没有 sl 这个寄存器名
movq %rax,$0x123			source 和 destination 搞反了
movl %eax,%rdx				这条没有错误
movb %si, 8(%rbp)			应该使用 movw 指令
```



### 3.4.3 移动数据举例

一个C语言函数：

```c
long exchange(long *xp, long y){
    long x = *xp;
    *xp = y;
    return x;
}
```

它对应的汇编语言版本：

```assembly
;long exchange(long *xp, long y)
;xp in %rdi, y in %rsi
exchange:
	movq (%rdi), %rax 	;Get x at xp. Set as return value.
	movq %rsi, (%rdi) 	;Store y at xp.
	ret Return.
```



### Practice Problem 3.4

Assume variables `sp` and `dp` are declared with types

```c
src_t *sp;
dest_t *dp;
```

where `src_t` and `dest_t` are data types declared with `typedef`. We wish to use the appropriate pair of data movement instructions to implement the operation

```c
*dp = (dest_t) *sp;
```

Assume that the values of `sp` and `dp` are stored in registers `%rdi` and `%rsi`, respectively. For each entry in the table, show the two instructions that implement the specified data movement. The **first ** instruction in the sequence should **read from memory**, do the appropriate conversion, and set the appropriate portion of register `%rax`. The **second** instruction should then write the appropriate portion of `%rax` to memory. In both cases, the portions may be `%rax`, `%eax`, `%ax`, or `%al`, and they may differ from one another.

Recall that when performing a cast that involves both a size change and a change of “signedness” in C, the operation should **change the size first** (Section 2.2.6).



| `src_t` | `dest_t` | Instruction |
| :-------: | :--------: | :-----------: |
| `long` | `long` | `movq (%rdi),%rax` |
|  |  | `movq %rax,(%rsi)` |
| `char` | `int` | `movsbl (%rdi),%eax` |
|  |  | `movl %eax,(%rsi)` |
| `char` | `unsigned` | `movsbl (%rdi),%eax` |
|  |  | `movl %eax,(%rsi)` |
| `unsigned char` | `long` | `movzbq (%rdi),%rax` |
|  |  | `movq %rax,(%rsi)` |
| `int` | `char` | `movl (%rdi),%eax` |
|  |  | `movb %al,(%rsi)` |
| `unsigned` | `unsigned char` | `movl (%rdi),%eax` |
|  |  | `movb %al,(%rsi)` |
| `char` | `short` | `movsbw (%rdi),%ax` |
|  |  | `movw %ax,(%rsi)` |



### Practice Problem 3.5

You are given the following information. A function with prototype

```C
void decode1(long *xp, long *yp, long *zp);
```

is compiled into assembly code, yielding the following:

```assembly
;void decode1(long *xp, long *yp, long *zp)
;xp in %rdi, yp in %rsi, zp in %rdx
decode1:
    movq (%rdi), %r8
    movq (%rsi), %rcx
    movq (%rdx), %rax
    movq %r8, (%rsi)
    movq %rcx, (%rdx)
    movq %rax, (%rdi)
    ret
```

Parameters `xp`, `yp`, and `zp` are stored in registers `%rdi`, `%rsi`, and `%rdx`, respectively.

Write C code for `decode1` that will have an effect equivalent to the assembly code shown.

```C
void decode1(long *xp, long *yp, long *zp){
    long x=*xp;
    long y=*yp;
    long z=*zp;
    
    *yp=x;
    *zp=y;
    *xp=z;
}
```



### 3.4.4 数据的进栈和出栈 pushing and popping

push and pop instructions:

{% asset_img Snipaste_2021-10-21_10-24-31.png %}

`%rsp` 寄存器存放当前**栈顶的地址**



## 3.5 算术和逻辑运算

### 3.5.1 加载有效地址

{% asset_img Snipaste_2021-10-21_20-41-50.png %}



### Practice Problem 3.6

Suppose register `%rbx` holds value p and `%rdx` holds value q. Fill in the table below with formulas indicating the value that will be stored in register `%rax` for each of the given assembly-code instructions:

|         Instruction         |       Result        |
| :-------------------------: | :-----------------: |
|    `leaq 9(%rdx), %rax`     |     `(%rdx)+9`      |
|  `leaq (%rdx,%rbx), %rax`   |   `(%rdx)+(%rbx)`   |
| `leaq (%rdx,%rbx,3), %rax`  |  `(%rdx)+3*(%rbx)`  |
|    `leaq 2(%rbx,%rbx,7)`    |    `8*(%rbx)+2`     |
|  `leaq 0xE(,%rdx,3), %rax`  |    `3*(%rdx)+14`    |
| `leaq 6(%rbx,%rdx,7), %rax` | `(%rbx)+7*(%rdx)+6` |



### Practice Problem 3.7

Consider the following code, in which we have omitted the expression being computed:

```c
short scale3(short x, short y, short z) {
	short t = ;
	return t;
}
```

Compiling the actual function with gcc yields the following assembly code:

```assembly
;short scale3(short x, short y, short z)
;x in %rdi, y in %rsi, z in %rdx
scale3:
    leaq (%rsi,%rsi,9), %rbx		;(%rbx)=10*y
    leaq (%rbx,%rdx), %rbx			;(%rbx)=10*y+z
    leaq (%rbx,%rdi,%rsi), %rbx		;(%rbx)=10*y+z+x*y=x*y+10*y+z
    ret
```

Fill in the missing expression in the C code.

```c
short scale3(short x, short y, short z) {
	short t = x*y+10*y+z;
	return t;
}
```



### 3.5.2 一元和二元操作

### Practice Problem 3.8

Assume the following values are stored at the indicated memory addresses and registers:

| address | value | register | value |
| :-----: | :---: | :------: | :---: |
|  0x100  | 0xFF  |   %rax   | 0x100 |
|  0x108  | 0xAB  |   %rcx   |  0x1  |
|  0x110  | 0x13  |   %rdx   |  0x3  |
|  0x118  | 0x11  |          |       |

Fill in the following table showing the effects of the following instructions, in terms of both the register or memory location that will be updated and the resulting value:

|        Instruction        | Destination |      Value      |
| :-----------------------: | :---------: | :-------------: |
|    `addq %rcx,(%rax)`     |    0x100    | 0xff+0x1=0x100  |
|    `subq %rdx,8(%rax)`    |    0x108    |  0xab-0x3=0xa8  |
| `imulq $16,(%rax,%rdx,8)` |    0x118    | 0x11*0x10=0x110 |
|      `incq 16(%rax)`      |    0x110    |      0x14       |
|        `decq %rcx`        |    %rcx     |        0        |
|     `subq %rdx,%rax`      |    %rax     |      0xFD       |



### 3.5.3 移位操作

### Practice Problem 3.9

Suppose we want to generate assembly code for the following C function:

```c
long shift_left4_rightn(long x, long n){
    x <<= 4;
    x >>= n;
    return x;
}
```

The code that follows is a portion of the assembly code that performs the actual shifts and leaves the final value in register `%rax`. Two key instructions have been omitted. Parameters **x** and **n** are stored in registers `%rdi ` and `%rsi`, respectively.

```assembly
;long shift_left4_rightn(long x, long n)
;x in %rdi, n in %rsi
shift_left4_rightn:
    movq %rdi, %rax ;Get x
    salq  $4,%rax	;x <<= 4
    movl %esi, %ecx ;Get n (4 bytes)
    sarq  %cl,%rax	;x >>= n
```

Fill in the missing instructions, following the annotations on the right. The right shift should be performed **arithmetically**.



### Practice Problem 3.10

Consider the following code, in which we have omitted the expression being computed:

```c
short arith3(short x, short y, short z){
    short p1 = ;
    short p2 = ;
    short p3 = ;
    short p4 = ;
    return p4;
}
```

The portion of the generated assembly code implementing these expressions is as follows:

书本上给的汇编代码有些错误，我按照自己编译出来的汇编代码进行了修改

```assembly
;short arith3(short x, short y, short z)
;x in %rdi, y in %rsi, z in %rdx
arith3:
    orq %rsi, %rdx		;z |= y
    sarq $9, %rdx		;z >>= 9
    notq %rdx			;z ~= z;
    movq %rsi, %rbx		;(%rbx)=y
    subq %rdx, %rbx		;y=y-z
    ret
```

```c
short arith3(short x, short y, short z){
    short p1 = y|z;
    short p2 = p1>>9;
    short p3 = ~p2;
    short p4 = y-p3;
    return p4;
}
```



### Practice Problem 3.11

It is common to find assembly-code lines of the form

```assembly
xorq %rcx,%rcx
```

in code that was generated from C where no exclusive-or operations were present.

A. Explain the effect of this particular exclusive-or instruction and what useful operation it implements.

将寄存器内容清零

B. What would be the more straightforward way to express this operation in assembly code?

```assembly
movq $0,%rcx
```

C. Compare the number of bytes to encode any two of these three different implementations of the same operation.

事实证明，使用异或置零生成的代码**体积更小**

### 3.5.5 特殊算术操作



## 3.6 控制

### 3.6.1 标志寄存器 Condition Codes

> **CF**: Carry flag. The most recent operation generated a carry out of the most significant bit. Used to detect overflow for **unsigned operations**.
>
> **ZF**: Zero flag. The most recent operation yielded zero. 
>
> **SF**: Sign flag. The most recent operation yielded a negative value. 
>
> **OF**: Overflow flag. The most recent operation caused a **two’s-complement overflow**—either negative or positive.



{% asset_img Snipaste_2021-10-22_20-32-34.png %}



### 3.6.2 获取标志的值 Accessing the Condition Codes

{% asset_img Snipaste_2021-10-22_20-39-32.png %}



### Practice Problem 3.13

The C code

```c
int comp(data_t a, data_t b) {
	return a COMP b;
}
```

shows a general comparison between arguments a and b, where `data_t`, the data type of the arguments, is defined (via `typedef`) to be one of the integer data types listed in Figure 3.1 and either signed or unsigned. The comparison `COMP ` is defined via `#define`.

Suppose a is in some portion of `%rdx` while b is in some portion of `%rsi`. For each of the following instruction sequences, determine which data types `data_t` and which comparisons `COMP` could cause the compiler to generate this code. (There can be multiple correct answers; you should **list them all**.)

A. 

```assembly
cmpl %esi, %edi
setl %al
```

| number | data type | operation |
| ------ | --------- | --------- |
| 1      | int       | a<b       |

B. 

```assembly
cmpw %si, %di
setge %al
```
| number | data type | operation |
| ------ | --------- | --------- |
| 1      | short     | a >= b    |



C. 

```assembly
cmpb %sil, %dil
setbe %al
```
| number | data type     | operation |
| ------ | ------------- | --------- |
| 1      | unsigned char | a <= b    |



D.

```assembly
cmpq %rsi, %rdi
setne %a
```
| number | data type            | operation |
| ------ | -------------------- | --------- |
| 1      | long                 | a != b    |
| 2      | unsigned long        | a != b    |
| 3      | some form of pointer | a != b    |



### Practice Problem 3.14

The C code

```c
int test(data_t a) {
	return a TEST 0;
}
```

shows a general comparison between argument a and 0, where we can set the data type of the argument by declaring `data_t` with a `typedef`, and the nature of the comparison by declaring TEST with a `#define` declaration. The following instruction sequences implement the comparison, where `a`  is held in some portion of register `%rdi`. For each sequence, determine which data types `data_t` and which comparisons TEST could cause the compiler to generate this code. (There can be multiple correct answers; **list all correct ones**.)

A. 

```assembly
testq %rdi, %rdi
setge %al
```

| number | data type | operation |
| ------ | --------- | --------- |
| 1      | long      | a >= 0    |

B. 

```assembly
testw %di, %di
sete %al
```

| number | data type      | operation |
| ------ | -------------- | --------- |
| 1      | short          | a == 0    |
| 2      | unsigned short | a == 0    |

C. 

```assembly
testb %dil, %dil
seta %al
```

| number | data type     | operation |
| ------ | ------------- | --------- |
| 1      | unsigned char | a > 0     |

D. 

```assembly
testl %edi, %edi 
setle %al
```

| number | data type | operation |
| ------ | --------- | --------- |
| 1      | int       | a <= 0    |



### 3.6.3 Jump 指令

contrived adj. 人为的；做作的；不自然的

{% asset_img Snipaste_2021-10-22_21-40-10.png %}



### 3.6.4 Jump 指令的编码

跳转指令的目标地址编码方式有两种：一种是 PC relative ，即给出目标地址相对于**即将要执行的指令**（当前指令的下一条）的相对位移；另一种是直接给出目标地址的绝对地址 absolute address



### Practice Problem 3.15

In the following excerpts from a disassembled binary, some of the information has been replaced by X’s. Answer the following questions about these instructions.

A. What is the target of the `je` instruction below? (You do not need to know anything about the `callq` instruction here.)

```assembly
4003fa: 74 02 		je XXXXXX
4003fc: ff d0 		callq *%rax
```

0x4003fc+0x02=0x4003fe



B. What is the target of the `je` instruction below?

```assembly
40042f: 74 f4 		je XXXXXX 
400431: 5d 			pop %rbp
```

0x400431-0xc=0x400425



C. What is the address of the `ja` and `pop` instructions?

```assembly
400543: 77 02 		ja 400547
400545: 5d 			pop %rbp
```



D. In the code that follows, the jump target is encoded in PC-relative form as a 4- byte two’s-complement number. The bytes are listed from least significant to most, reflecting the little-endian byte ordering of x86-64.What is the address of the jump target?

```assembly
4005e8: e9 73 ff ff ff jmpq XXXXXXX
4005ed: 90 nop
```

0xffff ff73= -141

0x4005ed - 141 = **0x400560**



### 3.6.5 使用条件控制指令实现条件分支

`cmp`  和 `jmp`  语句的结合



### Practice Problem 3.16

When given the C code

```c
void cond(short a, short *p){
    if (a && *p < a)
    *p = a;
}
```

gcc generates the following assembly code:

```assembly
;void cond(short a, short *p)
;a in %rdi, p in %rsi
cond:
    testq %rdi, %rdi
    je .L1
    cmpq %rdi, (%rsi)
    jle .L1
    movq %rdi, (%rsi)
.L1:
	rep; ret
```

A. Write a `goto` version in C that performs the same computation and mimics the control flow of the assembly code, in the style shown in Figure 3.16(b). You might find it helpful to first annotate the assembly code as we have done in our examples.

```c
void cond(short a,short *p){
    if(!a)
        goto s1;
    if(*p >= a)
        goto s1;
    *p=a;
  s1:
    return;
}
```



### Practice Problem 3.17

An alternate rule for translating `if` statements into goto code is as follows:

```c
    t = test-expr;
    if (t)
    	goto true;
    else-statement
    	goto done;
true:
    then-statement
done:
```



### Practice Problem 3.18

Starting with C code of the form

```c
short test(short x, short y, short z) {
    short val = y+z-x;
    if (z > 5) {
        if (y > 2)
            val = x/z;
        else
        	val = x/y;
    } else if (z < 3)
    	val = z/y;
    return val;
}
```

gcc generates the following assembly code:

```assembly
;short test(short x, short y, short z)
;x in %rdi, y in %rsi, z in %rdx
test:
    leaq (%rdx,%rsi), %rax
    subq %rdi, %rax
    cmpq $5, %rdx
    jle .L2
    cmpq $2, %rsi
    jle .L3
    movq %rdi, %rax
    idivq %rdx, %rax
    ret
.L3:
    movq %rdi, %rax
    idivq %rsi, %rax
    ret
.L2:
    cmpq $3, %rdx
    jge .L4
    movq %rdx, %rax
    idivq %rsi, %rax
.L4:
	rep; ret
```

Fill in the missing expressions in the C code.



### 3.6.6 使用条件移动（Conditional Moves）实现条件分支（Conditional Branches）

使用现代处理器时，Conditional Moves 方法通常会比使用 Conditional Control 更加高效



### Practice Problem 3.19

Running on a new processor model, our code required around **45 cycles** when the branching pattern was **random**, and around **25 cycles** when the pattern was **highly predictable**.

A. What is the approximate miss penalty?
$$
\left\{\begin{matrix}
     T_{ran}=T_{OK}+0.5*T_{MP}\\
     T_{High}=T_{OK}+0.1*T_{MP}
\end{matrix}\right.
\\
T_{OK}=20 \\
T_{MP}=50 \\
$$
50 个机器周期

B. How many cycles would the function require when the branch is mispredicted?

70 个机器周期



条件移动指令：

{% asset_img Snipaste_2021-11-01_21-54-30.png %}



### Practice Problem 3.20

In the following C function, we have left the definition of operation `OP` incomplete:

```c
#define OP 			/* Unknown operator */
short arith(short x) {
	return x OP 16;
}
```



When compiled, gcc generates the following assembly code:

```assembly
;short arith(short x)
;x in %rdi
arith:
    leaq 15(%rdi), %rbx		;temp=x+15
    testq %rdi, %rdi		;	
    cmovns %rdi, %rbx		;if(x>=0) temp=x	
    sarq $4, %rbx			;temp=temp>>4	
    ret
```

这是除法操作的汇编代码，所以题目中缺失的运算符是 `/`

**注意**当 x<0 时，由于使用2的补码表示的整型除以2的整数次幂的时候要加一个 bias （具体看看 2.3.7），所以才有第一句 `leaq 15(%rdi), %rbx		;temp=x+15` 

因为读第二章和第三章两章时间间隔有点久，这里被这个 +15 的操作难住了，记忆力不好



### Practice Problem 3.21

Starting with C code of the form

```c
short test(short x, short y) {
	short val = ;
	if ( ) {
		if ( )
			val = ;
		else
			val = ;
	} else if ( )
		val = ;
    
	return val;
}
```

gcc generates the following assembly code:

```assembly
;short test(short x, short y)
;x in %rdi, y in %rsi
test:
    leaq 12(%rsi), %rbx
    testq %rdi, %rdi
    jge .L2
    movq %rdi, %rbx
    imulq %rsi, %rbx ;val=x*y;
    movq %rdi, %rdx  ;temp=x
    orq %rsi, %rdx   ;temp=temp|y
    cmpq %rsi, %rdi  ;
    cmovge %rdx, %rbx;if(x>=y) val=x|y;
    ret
.L2:
    idivq %rsi, %rdi
    cmpq $10, %rsi
    cmovge %rdi, %rbx
    ret
```

Fill in the missing expressions in the C code.

```c
short test(short x, short y) {
	short val = y+12;
	if (x!=0) {
		if (x >= y)
			val = x|y;
		else
			val = x*y;
	} else if (y >= 10)
		val = x/y;
    
	return val;
}
```



### 3.6.7 循环

* `do-while` 循环

### Practice Problem 3.23

For the C code

```c
short dw_loop(short x) {
    short y = x/9;
    short *p = &x;
    short n = 4*x;
    do {
        x += y;
        (*p) += 5;
        n -= 2;
    } while (n > 0);
    return x;
}
```

gcc generates the following assembly code:

```assembly
;short dw_loop(short x)
;x initially in %rdi
dw_loop:
    movq %rdi, %rbx			;load x into rbx
    movq %rdi, %rcx			;load x into rcx
    idivq $9, %rcx			; y=x/9 y in rcx
    leaq (,%rdi,4), %rdx	; n=4*x n in rdx
.L2:
    leaq 5(%rbx,%rcx), %rcx	;x=x+y+5
    subq $2, %rdx			;n-=2
    testq %rdx, %rdx		;
    jg .L2					;if n>0 ,goto .L2
    rep; ret
```



A. Which registers are used to hold program values x, y, and n?

rdi and rcx

rcx

rdx

B. How has the compiler eliminated the need for pointer variable `p` and the pointer dereferencing implied by the expression `(*p)+=5` ?

C. Add annotations to the assembly code describing the operation of the program.



* `while` loops

### Practice Problem 3.24

For C code having the general form

```c
short loop_while(short a, short b){
    short result = ;
    while ( ) {
        result = ;
        a = ;
    }
    return result;
}
```

gcc, run with command-line option `-Og`, produces the following code:

```assembly
;short loop_while(short a, short b)
;a in %rdi, b in %rsi
loop_while:
    movl $0, %eax
    jmp .L2
.L3:
    leaq (,%rsi,%rdi), %rdx
    addq %rdx, %rax
    subq $1, %rdi
.L2:
    cmpq %rsi, %rdi
    jg .L3
    rep; ret
```

We can see that the compiler used a jump-to-middle translation, using the `jmp` instruction on line 3 to jump to the test starting with label `.L2`. Fill in the missing parts of the C code.

```c
short loop_while(short a, short b){
    short result = 0;
    while (a > b) {
        result = result + a*b;
        a = a - 1;
    }
    return result;
}
```



### Practice Problem 3.25

For C code having the general form

```c
long loop_while2(long a, long b){
    long result = ;
    while ( ) {
        result = ;
        b = ;
    }
    return result;
}
```

gcc, run with command-line option `-O1`, produces the following code:

```assembly
;a in %rdi, b in %rsi
loop_while2:
    testq %rsi, %rsi
    jle .L8
    movq %rsi, %rax
.L7:
    imulq %rdi, %rax
    subq %rdi, %rsi
    testq %rsi, %rsi
    jg .L7
    rep; ret
.L8:
	movq %rsi, %rax
	ret
```

We can see that the compiler used a **guarded-do** translation, using the `jle` instruction on line 3 to skip over the loop code when the initial test fails. Fill in the missing parts of the C code. Note that the control structure in the assembly code does not exactly match what would be obtained by a direct translation of the C code according to our translation rules. In particular, it has two different `ret` instructions (lines 10 and 13). However, you can fill out the missing portions of the C code in a way that it will have equivalent behavior to the assembly code.

```c
long loop_while2(long a, long b){
    long result = b;
    while (b > 0) {
        result = a*b;
        b = b - a;
    }
    return result;
}
```



### Practice Problem 3.26

A function `test_one` has the following overall structure:

```c
short test_one(unsigned short x) {
    short val = 1;
    while (x != 0) {
    	val = val ^ x;
        x >>= 1;
    }
    return val & 0x00;
}
```

The gcc C compiler generates the following assembly code:

```assembly
;short test_one(unsigned short x)
;x in %rdi
test_one:
    movl $1, %eax
    jmp .L5
.L6:
    xorq %rdi, %rax
    shrq %rdi Shift right by 1
.L5:
    testq %rdi, %rdi
    jne .L6
    andl $0, %eax
    ret
```

Reverse engineer the operation of this code and then do the following:

A. Determine what loop translation method was used.

jump-to-middle

B. Use the assembly-code version to fill in the missing parts of the C code.
C. Describe in English what this function computes.

经测试，书中给出的代码有错误。如果运行书中的代码，无论如何，返回值都是0。

把原汇编代码改成

```assembly
;short test_one(unsigned short x)
;x in %rdi
test_one:
    movl $1, %eax
    jmp .L5
.L6:
    xorq %rdi, %rax
    shrq %rdi Shift right by 1
.L5:
    testq %rdi, %rdi
    jne .L6
    andl $1, %eax			;把0改成1
    ret
```

相应的C代码改成

```C
short test_one(unsigned short x) {
    short val = 1;
    while (x != 0) {
    	val = val ^ x;
        x >>= 1;
    }
    return val & 1;
}
```

这样的代码就实现了答案中所说的计算 x 中1出现次数奇偶性的功能，出现奇数个1返回0，偶数个1返回1。

测试结果：

```SHELL
x=0x01,val=0
x=0x11,val=1
x=0x111,val=0
```



* `for` loops

`for` 循环在本质上和 `while` 循环是相同的。自然地，`for` 循环也有两种翻译成机器语言的方法 jump-to-middle & guarded-do

下面是一个计算阶乘的C语言函数

```c
long fact_for(long n){
    long i;
    long result = 1;
    for (i = 2; i <= n; i++)
    	result *= i;
    return result;
}
```

对应的 `while` 版本

```c
long fact_for_while(long n){
    long i = 2;
    long result = 1;
    while(i <= n){
        result *= i;
        i++;
    }
    return result;
}
```

jump-to-middle 版本机器语言对应的C语言（使用 goto）

```c
long fact_for_jm_goto(long n){
    long i = 2;
    long result = 1;
    goto test;
   loop:
    result *= i;
    i++;
   test:
    if(i<=n)
        goto loop;
    return result;
}
```



jump-to-middle 版本汇编语言

```assembly
fact_for:
	movl 	$1,%eax
	movl 	$2,%edx
	jmp 	.L8
.L9:
	imulq 	%rdx,%rax
	addq	$1,%rdx
.L8:
	cmpq 	%rdi,%rdx
	jle		.L9
	rep;ret
```



### Practice Problem 3.27 

Write `goto` code for a function called `fibonacci` to print fibonacci numbers using a `while` loop. Apply the guarded-do transformation.

```c
void fibonacci(int n){
    int num1 = 0;
    int num2 = 1;
    int counter = 0;
    int temp = 0;
    if(counter == n)
        goto done;
   loop:
    printf("%d\n",num2);
    temp = num2;
    num2 += num1;
    num1 = temp;
    counter++;
    if(counter<n)
        goto loop;
   done:
    return;
}
```



### Practice Problem 3.28

A function `test_two` has the following overall structure:

```c
short test_two(unsigned short x) {
    short val = 0;
    short i;
    for ( ... ; ... ; ... ) {
    ...
    }
    return val;
}
```

The gcc C compiler generates the following assembly code:

```assembly
;test fun_b(unsigned test x)
;x in %rdi
test_two:
    movl $1, %edx
    movl $65, %eax
.L10:
    movq %rdi, %rcx
    andl $1, %ecx
    addq %rax, %rax
    orq %rcx, %rax
    shrq %rdi ;Shift right by 1
    addq $1, %rdx
    jne	.L10
	rep; ret
```

Reverse engineer the operation of this code and then do the following:

A. Use the assembly-code version to fill in the missing parts of the C code. 

结合答案来看这个汇编代码，错误很多啊，为啥第三章代码的错误这么多，真的耽误时间

以下是答案给出的C源代码：

```c
long fun_b(unsigned long x) {
    long val = 0;
    long i;
    for (i = 64; i != 0; i--) {
        val = (val << 1) | (x & 0x1);
        x >>= 1;
    }
    return val;
}
```
使用以下命令编译

```shell
gcc -O1 -S .\p328.c
```

得到的汇编代码（简化后）：

```assembly
fun_b:
	.seh_endprologue
	movl	$64, %edx		;i=64
	movl	$0, %eax		;val=0
.L2:
	addl	%eax, %eax		;val+=val 或者 val <<= 1
	movl	%ecx, %r8d		;
	andl	$1, %r8d		;(x&1)
	orl	%r8d, %eax			;val |= (x&1)
	shrl	%ecx			;x >>= 1
	subl	$1, %edx		;i--
	jne	.L2
	ret
```

这样才能对应上，题目给的汇编代码错误很多

B. Explain why there is neither an initial test before the loop nor an initial jump to the test portion of the loop. 

C. Describe in English what this function computes. 

**bit-wise inverse**



### 3.6.8 `switch` 语句

### Practice Problem 3.30

In the C function that follows, we have omitted the body of the `switch` statement. In the C code, the case labels did not span a contiguous range, and some cases had multiple labels.

```c
void switch2(short x, short *dest) {
    short val = 0;
    switch (x) {
    	...Body of switch statement omitted
    }
    *dest = val;
}
```

In compiling the function, gcc generates the assembly code that follows for the initial part of the procedure, with variable `x` in `%rdi`:

```assembly
;void switch2(short x, short *dest)
;x in %rdi
switch2:
    addq $2, %rdi		;将x加2以确保最小的label等于0，所以最小的label是-2
    cmpq $8, %rdi		;
    ja .L2				;if x+2>8 goto .L2
    jmp *.L4(,%rdi,8)	;goto 
```

```assembly
.L4:
.quad .L9
.quad .L5
.quad .L6
.quad .L7
.quad .L2
.quad .L7
.quad .L8
.quad .L2
.quad .L5
```

Based on this information, answer the following questions:

A. What were the values of the case labels in the `switch` statement?

-2,-1,0,1,3,4,6

B. What cases had multiple labels in the C code?

1和3，-1和6



### Practice Problem 3.31

For a C function `switcher` with the general structure

```c
void switcher(long a, long b, long c, long *dest){
    long val;
    switch(a) {
        case 5: /* Case A */
            c = a^15;
            /* Fall through */
        case 0: /* Case B */
            val = c+112;
            break;
        case 2: /* Case C */
        case 7: /* Case D */
            val = (a+c)<<2;
            break;
        case 4: /* Case E */
            val = b;
            break;
        default:
        	val = a;
    }
    *dest = val;
}
```

```assembly
;void switcher(long a, long b, long c, long *dest)
;a in %rsi, b in %rdi, c in %rdx, d in %rcx
switcher:
    cmpq $7, %rdi
    ja .L2
    jmp *.L4(,%rdi,8)
    .section .rodata
  .L7:
    xorq $15, %rsi
    movq %rsi, %rdx
  .L3:
    leaq 112(%rdx), %rdi
    jmp .L6
  .L5:
    leaq (%rdx,%rsi), %rdi
    salq $2, %rdi
    jmp .L6
  .L2:
    movq %rsi, %rdi
  .L6:
    movq %rdi, (%rcx)
    ret
```

Jump table

```assembly
.L4:
    .quad .L3
    .quad .L2
    .quad .L5
    .quad .L2
    .quad .L6
    .quad .L7
    .quad .L2
    .quad .L5
```



## 3.7 过程 procedures

这里的过程是一种抽象概念，在不同的编程语言中，过程的具体形式有：函数、方法、子例程和处理函数等等

要提供对**过程**的机器级支持，必须处理好以下三件事：控制权的传递、数据（参数）的传递、内存的分配和**释放**

### 3.7.1 运行时栈 run-time stack

x86-64 的栈向**低地址方向**增长，所以**减小栈指针**实际上是**入栈**操作

x86-64 上栈的通用结构

{% asset_img Snipaste_2021-11-16_21-01-00.png %}

### 3.7.2 控制权的转移

### Practice Problem 3.32

The disassembled code for two functions `first` and `last` is shown below, along with the code for a call of `first` by function `main`:

```assembly
;Disassembly of last(long u, long v)
;u in %rdi, v in %rsi
0000000000400540 <last>:
    400540: 48 89 f8 mov %rdi,%rax 			;L1: u
    400543: 48 0f af c6 imul %rsi,%rax 		;L2: u*v
    400547: c3 retq 						;L3: Return
```

```assembly
;Disassembly of last(long x)
;x in %rdi
0000000000400548 <first>:
    400548: 48 8d 77 01 lea 0x1(%rdi),%rsi 			;F1: x+1
    40054c: 48 83 ef 01 sub $0x1,%rdi 				;F2: x-1
    400550: e8 eb ff ff ff callq 400540 <last> 		;F3: Call last(x-1,x+1)
    400555: f3 c3 repz retq 						;F4: Return
    ...
    400560: e8 e3 ff ff ff callq 400548 <first> 	;M1: Call first(10)
    400565: 48 89 c2 mov %rax,%rdx 					;M2: Resume
```

| Label | PC       | Instruction | %rdi | %rsi | %rax | %rsp           | *%rsp |  Description   |
| :-----: | :--------: | :-----------: | :---: | :---: | :----: | :--------------: | :-----: | :------------: |
| M1    | 0x400560 | callq       | 10   | -    | -    | 0x7fffffffe820 | -     | call first(10) |
| F1 | 0x400548 | lea | 10 | - | - | 0x7fffffffe818 | 0x400565 | lea 0x1(%rdi),%rsi |
| F2 | 0x40054c | sub | 10 | 11 | - | 0x7fffffffe818 | 0x400565 | sub $0x1,%rdi |
| F3 | 0x400550 | callq | 9 | 11 | - | 0x7fffffffe818 | 0x400565 | call last(9,11) |
| L1 | 0x400540 | mov | 9 | 11 | - | 0x7fffffffe810 | 0x400555 | mov %rdi,%rax |
| L2 | 0x400543 | imul | 9 | 11 | 9 | 0x7fffffffe810 | 0x400555 | imul %rsi,%rax |
| L3 | 0x400547 | retq | 9 | 11 | 99 | 0x7fffffffe810 | 0x400555 | retq |
| F4 | 0x400555 | repz retq | 9 | 11 | 99 | 0x7fffffffe818 | 0x400565 | repz retq |
| M2 | 0x400565 | mov | 9 | 11 | 99 | 0x7fffffffe820 | - | mov %rax,%rdx |



### 3.7.3 数据（参数）传递

x86-64中，可以通过**寄存器**最多传递6个**整型**参数

寄存器的使用是有特殊顺序的，寄存器使用的名字取决于要传递的数据类型的大小

{% asset_img Snipaste_2021-11-16_23-18-38.png %}

如果函数要传递的参数超过6个，则超过的部分就要用**栈**来传递

### Practice Problem 3.33

A C function `procprob` has four arguments u, a, v, and b. Each is either a signed number or a pointer to a signed number, where the numbers have different sizes. The function has the following body:

```C
*u += a;
*v += b;
return sizeof(a) + sizeof(b);
```

It compiles to the following x86-64 code:

```assembly
procprob:
    movslq %edi, %rdi
    addq %rdi, (%rdx)
    addb %sil, (%rcx)
    movl $6, %eax
	ret
```

Determine a valid ordering and types of the four parameters. There are two correct answers.

```c
procprob(int a,short b,long *u,char *v)
```

```c
procprob(int b,short a,long *v,char *u)
```



### 3.7.4 栈上的局部存储

一般的函数不会使用栈存储，但是以下几种情况则需要使用栈作为存储区域：

* 函数参数过多，寄存器不足以存放
* 对一个局部变量使用 '&' 取地址运算符，因为需要获得地址，而寄存器没有地址，所以需要使用栈
* 某些局部变量是结构或者数组，因此必须能够通过数组或结构被引用到，也需要存放在栈中

看一个例子：

```c
long call_proc(){
    long x1 = 1; int x2 = 2;
    short x3 = 3; char x4 = 4;
    proc(x1, &x1, x2, &x2, x3, &x3, x4, &x4);
    return (x1+x2)*(x3-x4);
}
```

对应的汇编代码

```assembly
long call_proc()
call_proc:
	;Set up arguments to proc
	subq $32, %rsp ;Allocate 32-byte stack frame
	movq $1, 24(%rsp) ;Store 1 in &x1
	movl $2, 20(%rsp) ;Store 2 in &x2
	movw $3, 18(%rsp) ;Store 3 in &x3
	movb $4, 17(%rsp) ;Store 4 in &x4
	leaq 17(%rsp), %rax ;Create &x4
	movq %rax, 8(%rsp) ;Store &x4 as argument 8
	movl $4, (%rsp) ;Store 4 as argument 7
	leaq 18(%rsp), %r9 ;Pass &x3 as argument 6
	movl $3, %r8d ;Pass 3 as argument 5
	leaq 20(%rsp), %rcx ;Pass &x2 as argument 4
	movl $2, %edx ;Pass 2 as argument 3
	leaq 24(%rsp), %rsi ;Pass &x1 as argument 2
	movl $1, %edi ;Pass 1 as argument 1
 	;Call proc
	call proc
	;Retrieve changes to memory
	movslq 20(%rsp), %rdx ;Get x2 and convert to long
	addq 24(%rsp), %rdx ;Compute x1+x2
	movswl 18(%rsp), %eax ;Get x3 and convert to int
	movsbl 17(%rsp), %ecx ;Get x4 and convert to int
	subl %ecx, %eax ;Compute x3-x4
	cltq ;Convert to long
	imulq %rdx, %rax ;Compute (x1+x2) * (x3-x4)
	addq $32, %rsp ;Deallocate stack frame
	ret ;Return
```

可以看到汇编代码的大部分都在为调用proc这个函数做准备

它先是将 stack pointer 减去32来开辟一块32字节大小的栈内存，然后依次存储x1，x2，x3，x4

接着将参数1到参数6保存到规定的寄存器，参数7和参数8则放在栈上（相对栈指针的偏移地址分别为0和8）

这个函数的栈帧如下图所示：

{% asset_img Snipaste_2021-11-17_02-11-00.png %}



### 3.7.5 寄存器中的局部存储空间

x86-64中的寄存器分为两种：被调用者保存寄存器和调用者保存寄存器



### 3.7.6 递归过程

递归函数的一个经典例子：阶乘函数

```c
long rfact(long n){
    long result;
    if (n <= 1)
    result = 1;
    else
    result = n * rfact(n-1);
    return result;
}
```

对应的汇编代码

```assembly
;long rfact(long n)
;n in %rdi
rfact:
    pushq %rbx 				;Save %rbx
    movq %rdi, %rbx 		;Store n in callee-saved register
    movl $1, %eax 			;Set return value = 1
    cmpq $1, %rdi 			;Compare n:1
    jle .L35 				;If <=, goto done
    leaq -1(%rdi), %rdi 	;Compute n-1
    call rfact 				;Call rfact(n-1)
    imulq %rbx, %rax 		;Multiply result by n
.L35: 						;done:
    popq %rbx 				;Restore %rbx
    ret 					;Return
```



### Practice Problem 3.35

For a C function having the general structure

```c
long rfun(unsigned long x) {
    if (x == 0)
    	return 0;
    unsigned long nx = x>>2;
    long rv = rfun(nx);
    return x+rv;
}
```

gcc generates the following assembly code:

```assembly
;long rfun(unsigned long x)
;x in %rdi
rfun:
    pushq %rbx
    movq %rdi, %rbx
    movl $0, %eax
    testq %rdi, %rdi
    je .L2
    shrq $2, %rdi
    call rfun
    addq %rbx, %rax
.L2:
    popq %rbx
    ret
```

A. What value does `rfun` store in the callee-saved register `%rbx`?

~~rv~~  x

B. Fill in the missing expressions in the C code shown above.


## 3.8 数组分配和访问

### 3.8.1 基本原则

```c
T A[N];
```

起始位置为 $x_A$ ，这个声明会分配长为 $L*N$ 的连续内存，$L$ 是类型 T 的大小

第一个元素被存放在 $x_A$ 地址处，第 $i$ 个元素存放在 $x_A+L*i$ 地址处



### Practice Problem 3.36

```c
int P[5];
short Q[2];
int **R[9];
double *S[10];
short *T[2];
```

Fill in the following table describing the element size, the total size, and the address of element `i` for each of these arrays.

| Array | Element size | Total size | Start address | Element i |
| ----- | ------------ | ---------- | ------------- | --------- |
| P     | 4            | 4*5=20     | $x_P$         | $x_P+4*i$ |
| Q     | 2            | 4          | $x_Q$         | $x_Q+2*i$ |
| R     | 8            | 72         | $x_R$         | $x_R+8*i$ |
|S|8|80|$x_S$|$x_S+8*i$|
|T|8|16|$x_T$|$x_T+8*i$|



### 3.8.2 指针运算

C语言允许对指针进行运算，运算结果会根据指针指向的类型进行伸缩。比如指针 $p$ 指向一个类型长度为 $L$ 的类型，那么 $p+i$ 的运算结果就是 $x_p+L*i$

现在假设我们有一个整型数组 `E` 和整型索引 `i` 分别存放在寄存器 %rdx 和 %rcx 中

 {% asset_img Snipaste_2021-11-17_14-30-30.png %}

注意图中两个高亮部分的区别，获取地址用的是 `leaq` 指令，获取地址处的内容用 `movl` 指令



### Practice Problem 3.37

Suppose $x_P$ , the address of short integer array $P$, and long integer index $i$ are stored in registers `%rdx` and `%rcx`, respectively. For each of the following expressions, give its type, a formula for its value, and an assembly-code implementation. The result should be stored in register `%rax` if it is a **pointer** and register element `%ax` if it has data type **short**.

| Expression | Type                 | Value            | Assembly code              |
| ---------- | -------------------- | ---------------- | -------------------------- |
| `P[1]`     | `short`              | M[$x_P+2*1$]     | movw 2(%rdx),%ax           |
| `P+3+i`    | ~~`long`~~ `short *` | $x_P+6+2*i$      | leaq 6(%rdx,%rcx,2),%rax   |
| `P[i*6-5]` | `short`              | M[$x_P+12*i-10$] | movw -10(%rdx,%rcx,12),%ax |
| `P[2]`     | `short`              | M[$x_P+2*2$]     | movw 4(%rdx),%ax           |
| `&P[i+2]`  | ~~`long`~~ `short *` | $x_P+2*i+4$      | leaq 4(%rdx,%rcx,2),%rax   |



### 3.8.3 嵌套的数组

就是数组的数组

嵌套数组的引用和一维数组的没什么不同



### Practice Problem 3.38

Consider the following source code, where `M` and `N` are constants declared with `#define`:

```c
long P[M][N];
long Q[N][M];
long sum_element(long i, long j) {
	return P[i][j] + Q[j][i];
}
```



```assembly
;long sum_element(long i, long j)
;i in %rdi, j in %rsi
sum_element:
    leaq 0(,%rdi,8), %rdx
    subq %rdi, %rdx
    addq %rsi, %rdx
    leaq (%rsi,%rsi,4), %rax
    addq %rax, %rdi
    movq Q(,%rdi,8), %rax
    addq P(,%rdx,8), %rax
    ret
```

Use your reverse engineering skills to determine the values of `M` and `N` based on this assembly code.

M=5,N=7



### 3.8.4 定长数组

对于定长数组的使用，编译器会自动进行很多优化，例如下面这个函数

```c
#define N 16
typedef int fix_matrix[N][N];

/* Compute i,k of fixed matrix product */
int fix_prod_ele (fix_matrix A, fix_matrix B, long i, long k) {
    long j;
    int result = 0;
    for (j = 0; j < N; j++)
    	result += A[i][j] * B[j][k];
    return result;
}
```

优化后的C代码

```c
/* Compute i,k of fixed matrix product */
int fix_prod_ele_opt(fix_matrix A, fix_matrix B, long i, long k) {
    int *Aptr = &A[i][0]; /* Points to elements in row i of A */
    int *Bptr = &B[0][k]; /* Points to elements in column k of B */
    int *Bend = &B[N][k]; /* Marks stopping point for Bptr */
    int result = 0;
    do { 							/* No need for initial test */
        result += *Aptr * *Bptr; 	/* Add next product to sum */
        Aptr ++; 					/* Move Aptr to next column */
        Bptr += N; 					/* Move Bptr to next row */
    } while (Bptr != Bend); 		/* Test for stopping point */
    return result;
}
```

对应的汇编代码

```assembly
;int fix_prod_ele_opt(fix_matrix A, fix_matrix B, long i, long k)
;A in %rdi, B in %rsi, i in %rdx, k in %rcx
fix_prod_ele:
    salq $6, %rdx 						;Compute 64 * i
    addq %rdx, %rdi 					;Compute Aptr = xA + 64i = &A[i][0]
    leaq (%rsi,%rcx,4), %rcx 			;Compute Bptr = xB + 4k = &B[0][k]
    leaq 1024(%rcx), %rsi 				;Compute Bend = xB + 4k + 1024 = &B[N][k]
    movl $0, %eax 						;Set result = 0
.L7: 									;loop:
    movl (%rdi), %edx 					;Read *Aptr
    imull (%rcx), %edx 					;Multiply by *Bptr
    addl %edx, %eax 					;Add to result
    addq $4, %rdi 						;Increment Aptr ++
    addq $64, %rcx 						;Increment Bptr += N
    cmpq %rsi, %rcx 					;Compare Bptr:Bend
    jne .L7 							;If !=, goto loop
    rep; ret 							;Return
```



### Practice Problem 3.40

The following C code sets the diagonal elements of one of our fixed-size arrays to `val`:

```c
/* Set all diagonal elements to val */
void fix_set_diag(fix_matrix A, int val) {
    long i;
    for (i = 0; i < N; i++)
    	A[i][i] = val;
}
```

When compiled with optimization level `-O1`, gcc generates the following assembly code:

```assembly
fix_set_diag:
;void fix_set_diag(fix_matrix A, int val)
;A in %rdi, val in %rsi
	movl $0, %eax
.L13:
    movl %esi, (%rdi,%rax)
    addq $68, %rax
    cmpq $1088, %rax
    jne .L13
    rep; ret
```

Create a C code program `fix_set_diag_opt` that uses optimizations similar to those in the assembly code, in the same style as the optimized code in 3.8.4. Use expressions involving the parameter `N` rather than integer constants, so that your code will work correctly if `N` is redefined.

```c
#define N 16
typedef int fix_matrix[N][N];

void fix_set_diag_opt(fix_matrix A,int val){
    int *Aptr = A;
    int *Aend = &A[N][N];
    
    do{
        *Aptr = val;
        Aptr += (N+1);
    }while(Aptr != Aend)
}
```



## 3.9 异质（Heterogeneous）的数据结构

将不同类型对象组合到一起创建的数据类型

C语言中有两种这样的数据结构：struct 和 union

### 3.9.1 结构 strcut

几个重要的点：

* 结构的所有组成部分都存放在内存中一段**连续**的区域内
* 指向结构的指针就是结构第一个字节的地址（和数组类似）
* 编译器维护关于每个结构的信息，记录每个字段的字节偏移，从而可以使用结构指针和偏移引用结构元素



### Practice Problem 3.41

Consider the following structure declaration:

```c
struct test {
    short *p;
    struct {
        short x;
        short y;
    } s;
    struct test *next;
};
```

The following procedure (with some expressions omitted) operates on this structure:

```c
void st_init(struct test *st) {
    st->s.y = st->s.x;
    st->p = &(st->s.y);
    st->next = st;
}
```

A. What are the offsets (in **bytes**) of the following fields?

p : 0

s.x : 8

s.y : 10

next : 12

B. How many total bytes does the structure require?

8+2*2+8=20

C. The compiler generates the following assembly code for `st_init`:

```assembly
;void st_init(struct test *st)
;st in %rdi
st_init:
    movl 8(%rdi), %eax
    movl %eax, 10(%rdi)
    leaq 10(%rdi), %rax
    movq %rax, (%rdi)
    movq %rdi, 12(%rdi)
    ret
```

On the basis of this information, fill in the missing expressions in the code for `st_init`.



### 3.9.2 unions

联合提供了一种规避C语言类型系统的方法，允许使用多种类型来引用一个对象

联合还可以用来访问不同数据类型的位模式。例如，假设我们简单地将一个 `double` 类型的值 `d` 转换为 `unsigned long` 类型的 `u`

```c
unsigned long u = (unsigned long) d;
```

`u` 将会是 `d` 的整数部分。除了 d=0.0 之外，`u` 和 `d` 的位表示会很不一样

再看下面的代码

```C
unsigned long double2bits(double d) {
    union {
        double d;
        unsigned long u;
    } temp;
    temp.d = d;
    return temp.u;
};
```

以 `double` 类型存储 `d` ，以 `unsigned long` 类型返回它，就实现了**不改变位模式**的类型转换

### Practice Problem 3.43

Suppose you are given the job of checking that a C compiler generates the proper code for structure and union access. You write the following structure declaration:

```c
typedef union {
    struct {
        long u;
        short v;
        char w;
    } t1;
    struct {
        int a[2];
        char *p;
    } t2;
} u_type;
```

You write a series of functions of the form

```c
void get(u_type *up, type *dest) {
	*dest = expr;
}
```

with different access expressions `expr` and with destination data type *type* set according to type associated with `expr`. You then examine the code generated when compiling the functions to see if they match your expectations.

Suppose in these functions that `up` and `dest` are loaded into registers `%rdi` and `%rsi`, respectively. Fill in the following table with data type `type` and sequences of one to three instructions to compute the expression and store the result at `dest`.

|        expr        |  type  |                            Code                            |
| :----------------: | :----: | :--------------------------------------------------------: |
|      up->t1.u      |  long  |             movq (%rdi),%rax  movq %rax,(%rsi)             |
|      up->t1.v      | short  |              movw 8(%rdi),%ax movw %ax,(%rsi)              |
|     &up->t1.w      | char * |                    leaq 10(%rdi),(%rsi)                    |
| up->t2.a[up->t1.u] |  int   | movq (%rdi),%rax movl 11(%rdi,%rax,4),%eax mov %eax,(%rsi) |



### 3.9.3 数据对齐

数据对齐是提高内存操作性能的一种方法。

举个例子，一个处理器每次只能从内存地址为8的倍数的地方一次性读取8个字节，如果我们能保证所有double型的数据都保存在以8的倍数地址为起始地址的地方，那么处理器只需一个内存读取操作就能获得这个double型数据，否则需要两次。

数据对齐的规则：

$K$ 字节大小的数据的起始地址必须是 $K$ 的倍数

{% asset_img Snipaste_2021-11-18_14-16-41.png %}

### Practice Problem 3.44

For each of the following structure declarations, determine the offset of each field, the total size of the structure, and its alignment requirement for x86-64:

A. `struct P1 { short i; int c; int *j; short *d; };`

最小22字节，偏移分别是0，2，6，14

要求数据对齐：大小24字节，偏移分别是0，4，8，16，结构的起始地址需要是4的倍数

C. `struct P3 { long w[2]; int *c[2] };`

最小32字节，偏移分别是0，8，16，24

要求数据对齐：大小32字节，起始地址需要是8的倍数

D. `struct P4 { char w[16]; char *c[2] };`

最小32字节，偏移地址为0，16

对齐要求：起始地址是8的倍数

E.` struct P5 { struct P4 a[2]; struct P1 t };`

最小86字节

对齐要求：大小88字节，起始地址是8的倍数



## 3.10 在机器级程序中将控制与数据结合起来

### 3.10.1 理解指针

一些指针和机器码映射的关键原则

* 每个指针都对应一个类型
* 每个指针都有一个值

* 指针可以指向函数



### 3.10.2 使用GDB调试器

{% asset_img 308.png %}

### 3.10.3 内存越界引用和缓冲区溢出

大名鼎鼎的蠕虫就是利用缓冲区溢出进行攻击的

### Practice Problem 3.46

The code below shows a (low-quality) implementation of a function that reads a line from standard input, copies the string to newly allocated storage, and returns a pointer to the result.

```c
/* This is very low-quality code.
It is intended to illustrate bad programming practices.
See Practice Problem 3.46. */
char *get_line(){
    char buf[4];
    char *result;
    gets(buf);
    result = malloc(strlen(buf));
    strcpy(result, buf);
    return result;
}
```

Disassembly up through call to `gets`

```assembly
;char *get_line()
0000000000400720 <get_line>:
    400720: 53 				push %rbx
    400721: 48 83 ec 10 	sub $0x10,%rsp
;Diagram stack at this point
    400725: 48 89 e7 		mov %rsp,%rdi
    400728: e8 73 ff ff ff 	callq 4006a0 <gets>
;Modify diagram to show stack contents at this point
```

Consider the following scenario. Procedure `get_line` is called with the return address equal to 0x400076 and register `%rbx` equal to 0x0123456789ABCDEF. You type in the string

```shell
0123456789012345678901234
```

The program terminates with a segmentation fault. You run gdb and determine that the error occurs during the execution of the `ret` instruction of `get_line`.

A. Fill in the diagram that follows, indicating as much as you can about the stack just **after executing** the instruction at line 4 in the disassembly. Label the quantities stored on the stack (e.g., “Return address”) on the right, and their hexadecimal values (if known) within the box. Each box represents 8 bytes. Indicate the position of `%rsp`. Recall that the ASCII codes for characters 0–9 are 0x30–0x39.

|                         |                |
| ----------------------- | -------------- |
| 00 00 00 00 00 40 00 76 | Return address |
| 01 23 45 67 89 AB CD EF | %rbx           |
| -                       |                |
| -                       | %rsp           |

B. Modify your diagram to show the effect of the call to gets (line 7).

|                         |                |
| ----------------------- | -------------- |
| 00 00 00 00 00 40 00 34 | Return address |
| 33 32 31 30 39 38 37 36 | %rbx           |
| 35 34 33 32 31 30 39 38 |                |
| 37 36 35 34 33 32 31 30 | %rsp           |

C. To what address does the program attempt to return?

0x40 00 34

D. What register(s) have corrupted value(s) when get_line returns?

%rbx

E. Besides the potential for buffer overflow, what two other things are wrong with the code for `get_line`?

```shell
result = malloc(strlen(buf)+1);
```

应该检查 `malloc` 返回 `NULL` 的情况



### 3.10.4 对抗缓冲区溢出攻击

* 栈随机化

  依然可以使用 nop sled 空操作雪橇攻击

### Practice Problem 3.47

Running our stack-checking code 10,000 times on a system running Linux version 2.6.16, we obtained addresses ranging from a minimum of 0xffffb754 to a maximum of 0xffffd754.

A. What is the approximate range of addresses?

$2^{13}$

B. If we attempted a buffer overrun with a 128-byte nop sled, about how many attempts would it take to test all starting addresses?

$2^6=64$

* 栈破坏检测

  利用缓冲区溢出漏洞进行攻击的方法要改变栈上的内容，那么我们可以先在栈上存放一个值（金丝雀值 canary value），程序返回后再检查这个值是不是改变了，如果改变了，说明程序对栈上缓冲区以外的内存进行了更改，就可以马上终止程序。金丝雀值和栈上其他内容的排列方式如下图

  {% asset_img Snipaste_2021-11-19_01-43-41.png %}

### Practice Problem 3.48

The functions `intlen`, `len`, and `iptoa` provide a very convoluted way to compute the number of decimal digits required to represent an integer. We will use this as a way to study some aspects of the gcc stack protector facility.

```c
int len(char *s) {
	return strlen(s);
}

void iptoa(char *s, long *p) {
    long val = *p;
    sprintf(s, "%ld", val);
}

int intlen(long x) {
    long v;
    char buf[12];
    v = x;
    iptoa(buf, &v);
    return len(buf);
}
```

The following show portions of the code for `intlen`, compiled both **with** and **without** stack protector:

```assembly
;(a) Without protector
;int intlen(long x)
;x in %rdi
intlen:
    subq $40, %rsp
    movq %rdi, 24(%rsp)
    leaq 24(%rsp), %rsi
    movq %rsp, %rdi
    call iptoa
    
;(b) With protector
;int intlen(long x)
;x in %rdi
intlen:
    subq $56, %rsp
    movq %fs:40, %rax
    movq %rax, 40(%rsp)
    xorl %eax, %eax
    movq %rdi, 8(%rsp)
    leaq 8(%rsp), %rsi
    leaq 16(%rsp), %rdi
    call iptoa
```

A. For both versions: What are the positions in the stack frame for `buf`, `v`, and (when present) the canary value?

(a) buf :(%rsp) ,v:24(%rsp)

(b) buf :16(%rsp) ,v: 8(%rsp) , canary value: 40(%rsp)



* 限制可执行代码区域

  

### 3.10.5 支持变长栈帧

有一些函数需要使用变长数组，因此编译器不能提前计算出该函数需要的栈帧的大小

看下面这个函数

```c
long vframe(long n, long idx, long *q) {
    long i;
    long *p[n];
    p[0] = &i;
    for (i = 1; i < n; i++)
    	p[i] = q;
    return *p[idx];
}
```

在这个函数内部声明了一个长为 `n` 的 `long *` 数组，这需要 $8*n$ 个字节长度的栈内存

函数内部对 `i` 进行了取地址的操作，所以 `i` 也需要放在栈上

一起看看这个函数对应的汇编代码

```assembly
;long vframe(long n, long idx, long *q)
;n in %rdi, idx in %rsi, q in %rdx
;Only portions of code shown
vframe:
    pushq %rbp 				;Save old %rbp
    movq %rsp, %rbp 		;Set frame pointer
    subq $16, %rsp 			;Allocate space for i (%rsp = s1)
    leaq 22(,%rdi,8), %rax
    andq $-16, %rax
    subq %rax, %rsp 		;Allocate space for array p (%rsp = s2)
    leaq 7(%rsp), %rax
    shrq $3, %rax
    leaq 0(,%rax,8), %r8 	;Set %r8 to &p[0]
    movq %r8, %rcx 			;Set %rcx to &p[0] (%rcx = p)
	. . .
;Code for initialization loop
;i in %rax and on stack, n in %rdi, p in %rcx, q in %rdx
.L3: loop:
    movq %rdx, (%rcx,%rax,8) Set p[i] to q
    addq $1, %rax Increment i
    movq %rax, -8(%rbp) Store on stack
.L2:
    movq -8(%rbp), %rax Retrieve i from stack
    cmpq %rdi, %rax Compare i:n
    jl .L3 If <, goto loop
	. . .
;Code for function exit
    leave 				;Restore %rbp and %rsp
    ret 				;Return
```



## 3.11 浮点代码

为了处理浮点数，intel 和 AMD 都专门增加了一些指令和寄存器，这些寄存器称为媒体寄存器（media registers）

{% asset_img Randal E. Bryant, David R. O’Htive [3rd ed.] (2016, Pearson) 323.png %}



### 3.11.1 浮点传送和转换操作

浮点数移动指令：

{% asset_img Snipaste_2021-11-19_14-09-17.png %}

双操作数浮点转换指令：

{% asset_img Snipaste_2021-11-19_14-16-55.png %}

三操作数浮点转换指令：

{% asset_img Snipaste_2021-11-19_14-23-35.png %}



C函数

```c
double fcvt(int i, float *fp, double *dp, long *lp){
    float f = *fp; double d = *dp; long l = *lp;
    *lp = (long) d;
    *fp = (float) i;
    *dp = (double) l;
    return (double) f;
}
```

对应的汇编代码：

```assembly
;double fcvt(int i, float *fp, double *dp, long *lp)
;i in %edi, fp in %rsi, dp in %rdx, lp in %rcx
fcvt:
    vmovss (%rsi), %xmm0 					;Get f = *fp
    movq (%rcx), %rax 						;Get l = *lp
    vcvttsd2siq (%rdx), %r8 				;Get d = *dp and convert to long
    movq %r8, (%rcx) 						;Store at lp
    vcvtsi2ss %edi, %xmm1, %xmm1 			;Convert i to float
    vmovss %xmm1, (%rsi) 					;Store at fp
    vcvtsi2sdq %rax, %xmm1, %xmm1 			;Convert l to double
    vmovsd %xmm1, (%rdx) 					;Store at dp
    ;The following two instructions convert f to double
    vunpcklps %xmm0, %xmm0, %xmm0
    vcvtps2pd %xmm0, %xmm0
    ret 									;Return f
```



### Practice Problem 3.50

For the following C code, the expressions `val1`–`val4` all map to the program values `i`, `f`, `d`, and `l`:

```c
double fcvt2(int *ip, float *fp, double *dp, long l){
    int i = *ip; float f = *fp; double d = *dp;
    *ip = (int) val1;
    *fp = (float) val2;
    *dp = (double) val3;
    return (double) val4;
}
```

Determine the mapping, based on the following x86-64 code for the function:

```assembly
;double fcvt2(int *ip, float *fp, double *dp, long l)
;ip in %rdi, fp in %rsi, dp in %rdx, l in %rcx
;Result returned in %xmm0
fcvt2:
    movl (%rdi), %eax				;i=*ip
    vmovss (%rsi), %xmm0			;f=*fp
    vcvttsd2si (%rdx), %r8d			;d=*dp
    movl %r8d, (%rdi)				;*ip=(int)d
    vcvtsi2ss %eax, %xmm1, %xmm1	;
    vmovss %xmm1, (%rsi)			;*fp=(float)i
    vcvtsi2sdq %rcx, %xmm1, %xmm1	;
    vmovsd %xmm1, (%rdx)			;*dp=(double)l
    vunpcklps %xmm0, %xmm0, %xmm0	;
    vcvtps2pd %xmm0, %xmm0			;f=(double)f
    ret
```

val1->d

val2->i

val3->l

val4->f

### Practice Problem 3.51

The following C function converts an argument of type `src_t` to a return value of type `dst_t`, where these two types are defined using `typedef`:

```c
dest_t cvt(src_t x){
    dest_t y = (dest_t) x;
    return y;
}
```

For execution on x86-64, assume that argument `x` is either in `%xmm0` or in the appropriately named portion of register `%rdi` (i.e., %rdi or %edi). One or two instructions are to be used to perform the type conversion and to copy the value to the appropriately named portion of register `%rax` (integer result) or `%xmm0` (floating-point result). Show the instruction(s), including the source and destination registers.

| $T_x$  | $T_y$  |                  Instruction(s)                  |
| :----: | :----: | :----------------------------------------------: |
|  long  | double |           vcvtsi2sdq %rdi, %xmm0,%xmm0           |
| double |  int   |              vcvttsd2si %xmm0,%eax               |
| double | float  | vmovddup %xmm0,%xmm0<br />vcvtpd2psx %xmm0,%xmm0 |
|  long  | float  |           vcvtsi2ssq %rdi,%xmm0,%xmm0            |
| float  |  long  |              vcvttss2siq %xmm0,%rax              |



### 3.11.2 过程（函数）中的浮点代码

x86-64中使用XMM寄存器向函数传递浮点参数，以及从函数返回浮点值

{% asset_img 323.png %}

 从上图中看到，

* 可以使用 xmm0 ~ xmm7 传递8个浮点参数（超过8个的浮点参数使用栈传递）
* xmm0 返回浮点值
* 所有 XMM 寄存器都是**调用者**保存



### 3.11.3 浮点运算操作

{% asset_img Snipaste_2021-11-19_15-10-27.png %}

这些指令需要一个或两个操作数，第一个操作数可以是一个 XMM 寄存器或者内存位置，第二个操作数和目的操作数都**必须是** XMM 寄存器。

```c
double funct(double a, float x, double b, int i){
	return a*x - b/i;
}
```

对应的汇编代码

```assembly
;double funct(double a, float x, double b, int i)
;a in %xmm0, x in %xmm1, b in %xmm2, i in %edi
funct:
    ;The following two instructions convert x to double
    vunpcklps %xmm1, %xmm1, %xmm1
    vcvtps2pd %xmm1, %xmm1
    vmulsd %xmm0, %xmm1, %xmm0 		;Multiply a by x
    vcvtsi2sd %edi, %xmm1, %xmm1 	;Convert i to double
    vdivsd %xmm1, %xmm2, %xmm2 		;Compute b/i
    vsubsd %xmm2, %xmm0, %xmm0 		;Subtract from a*x
    ret 							;Return
```

### Practice Problem 3.54

Function `funct2` has the following prototype:

```c
double funct2(double w, int x, float y, long z);
```

Gcc generates the following code for the function:

```assembly
;double funct2(double w, int x, float y, long z)
;w in %xmm0, x in %edi, y in %xmm1, z in %rsi
funct2:
    vcvtsi2ss %edi, %xmm2, %xmm2		;
    vmulss %xmm1, %xmm2, %xmm1			;y *= x
    vunpcklps %xmm1, %xmm1, %xmm1		;
    vcvtps2pd %xmm1, %xmm2				;%xmm2 = (double)y
    vcvtsi2sdq %rsi, %xmm1, %xmm1		;y = (double)z
    vdivsd %xmm1, %xmm0, %xmm0			;w=w/z;
    vsubsd %xmm0, %xmm2, %xmm0			;w=y-w/z
    ret
```

Write a C version of `funct2`.

```c
double funct2(double w, int x, float y, long z){
    y *= x;
    return y-w/z;
}
```



### 3.11.4 定义和使用浮点常数

和整数操作不同的是，AVX浮点操作**不能**以**立即数**作为操作数，所以编译器必须为所有的浮点常数分配并初始化内存。

```C
double cel2fahr(double temp){
	return 1.8 * temp + 32.0;
}
```



```assembly
;double cel2fahr(double temp)
;temp in %xmm0
cel2fahr:
    vmulsd .LC2(%rip), %xmm0, %xmm0 	;Multiply by 1.8
    vaddsd .LC3(%rip), %xmm0, %xmm0 	;Add 32.0
	ret
.LC2:
    .long 3435973837 					;Low-order 4 bytes of 1.8
    .long 1073532108 					;High-order 4 bytes of 1.8
.LC3:
    .long 0 							;Low-order 4 bytes of 32.0
    .long 1077936128 					;High-order 4 bytes of 32.0
```

1.8 的表示：

高4字节 1073532108 (0x3ffccccc)

指数部分 

$e=0x3ff = 1023$  

$bias=2^{k-1}-1=2^{10}-1=1023$

$E=e-bias=0$

低4字节 3435973837 (0xcccccccd)

 将高4字节剩下的部分和低4字节组合起来 0xccccccccccccd，二进制小数为$f=0.8$

所以最后的浮点值为 $(1+0.8)*2^{0}=1.8$



 ### Practice Problem 3.55

Show how the numbers declared at label `.LC3` encode the number 32.0.

低4字节 0

高4字节 1077936128 (0x4040 0000) 

$e=0x404=1028$

$E=e-1023=5$

小数部分为0

所以最后的浮点值为 $(1+0.0)*2^5=32.0$



### 3.11.5 在浮点代码中使用位级操作

{% asset_img Snipaste_2021-11-20_00-33-32.png %}

### 3.11.6 浮点比较操作

{% asset_img Snipaste_2021-11-20_00-34-45.png %}
