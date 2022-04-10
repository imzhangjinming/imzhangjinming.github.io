---
title: 深入理解计算机系统 第七章 链接
date: 2021-12-03 02:28:00
categories:
- notes
tags:
---

第七章 链接



<!--more-->

# 第七章 链接

链接 (linking) 是将各种**代码**和**数据片段**收集并组合成为一个单一文件的过程，这个文件可被加载到内存并执行 。

链接器在软件开发中扮演着一个关键的角色，因为它们使得分离编译 (separate compilation)成为可能



## 7.1 编译器驱动程序 compiler driver

大多数编译系统提供编译器驱动程序, 它代表用户在需要时调用语言预处理器、编译器、汇编器和链接器。

```c
//main.c
int sum(int *a, int n);

int array[2] = {1, 2};

int main(){
	int val = sum(array, 2);
	return val;
}
```

```C
//sum.c
int sum(int *a, int n)
{
	int i, s = 0;

    for (i = 0; i < n; i++) {
        s += a[i];
    }
	return s;
}
```



使用下面这条命令编译这个程序：

```shell
linux> gcc -Og -o prog main.c sum.c
```

在这个过程中，gcc 先后为我们调用了 C 预处理器 (cpp)、C 编译器 (cc1)、汇编器 (as) 和链接器 (ld)

依次产生的中间文件为：
$$
main.c \rightarrow^{cpp} main.i \rightarrow^{cc1} main.s \rightarrow^{as} main.o \rightarrow^{ld} prog(executable )
$$


## 7.2 静态链接

像 Linux LD 程序这样的静态链接器 (static linker) 以一组可重定位目标文件和命令行参数作为输入，生成一个**完全链接**的、可以**加载和运行**的**可执行目标文件**作为输出。

为了构造可执行文件，链接器必须完成两个主要任务:

* **符号解析** (symbol resolution) 。目标文件**定义**和**引用**符号，每个符号对应于一个函数、一个全局变量或一个静态变量。符号解析的目的是**将每个符号引用正好和一个符号定义**关联起来。

* **重定位** (relocation) 。编译器和汇编器生成从地址 0 开始的代码和数据节。链接器通 过把每个符号定义与一个内存位置关联起来，从而重定位这些节，然后修改所有对 这些符号的引用，使得它们指向这个内存位置 。 



## 7.3 目标文件

目标文件有三种形式:

* **可重定位目标文件**。包含二进制代码和数据，其形式可以在编译时与其他可重定位 目标文件合并起来，创建一个可执行目标文件 。
* **可执行目标文件**。包含二进制代码和数据，其形式可以被直接复制到内存并执行 。
* **共享目标文件**。一种特殊类型的可重定位目标文件，可以在加载或者运行时被动态地加载进内存并链接。

从技术上来说， 一个目标模块 (object module)就是一个字节序列，而一个目标文件 （object file)就是一个以文件形式存放在磁盘中的目标模块 。

目标文件是按照特定的目标文件格式来组织的，各个系统的目标文件格式都不相同：

* Windows ： Portable Executable, PE
* Mac OS-X ： Mach-O
* 现代 x86-64 Linux 和 Unix 系统 ：Execut­able and Linkable Format, **ELF**



## 7.4 可重定位目标文件 

下面这张图片展示了一个典型的ELF可重定位目标文件的格式：

{% asset_img Snipaste_2021-12-02_12-56-25.png %}



### 7.6.2 与静态库链接

把一些目标模块打包成一个文件，就是静态库（static library）了。静态库可以用作链接器的输入

在linux环境下，静态库的后缀是 .a ，a是archive的首字母，指示静态库是一种归档文件

可以使用 AR 工具制作静态库



### 7.6.3 链接器如何使用静态库来解析引用

在符号解析阶段，链接器按照静态库在命令行上出现的顺序从左到右扫描

但是gcc的算法导致我们必须非常注意库在命令行中的位置，简单来说就是：定义必须放在引用之后

比如说：x 库引用了 y 库，那么命令行中就必须这样写：

```shell
linux> gcc foo.c libx.a liby.a
```

只有当引用外部符号的库出现在先，链接器才会带着寻找符号定义的目标扫描后面的库。



#### Practice Problem 7.3

a 和 b 表示当前目录中的目标模块或者静态库，而 a$\rightarrow$b 表示 a 依赖于 b, 也就是说 b 定义了一个被 a 引用的符号。对于下面每种场景，请给出最小的命令行(即一个 含有最少数量的目标文件和库参数的命令)，使得静态链接器能解析所有的符号引用。

A. p.o→libx.a

```shell
linux> ld p.o libx.a
```

B. p.o→libx.a→liby.a

```shell
linux> ld p.o libx.a liby.a
```

C. p.o→libx.a→liby.a and liby.a→libx.a→p.o

```shell
linux> ld p.o libx.a liby.a libx.a p.o
```



## 7.7 重定位

完成符号解析之后，链接器就知道它的输入目标模块中的代码节和数据节的确切大小。

接下来就开始重定位了。这由两步组成：

* 重定位节和符号定义

	在这一步中，链接器将所有相同类型的节合并为同一类型的**新的聚合节**。然后，链接器将**运行时内存地址**赋给新的聚合节，赋给输入模块定义的每个节，以及赋给输入模块定义的每个符号。

* 重定位节中的符号引用

	这一步中，链接器修改代码节和数据节中的符号引用，使它们指向正确的运行时地址。



### 7.7.1 重定位条目 relocation entry

重定位条目是汇编器生成的，它告诉链接器如何重定位位置未知的引用。

ELF重定位条目的格式如下：

```c
typedef struct {
    long offset; /* Offset of the reference to relocate */
    long type:32, /* Relocation type */
    symbol:32; /* Symbol table index */
    long addend; /* Constant part of relocation expression */
} Elf64_Rela;
```

`offset` 节内偏移量

`symbol` 引用指向的符号

`type` 引用的类型



两种最基本的重定位类型 上面的 `type`

* R_X86_64_PC32

	重定位一个使用32位PC**相对地址**的引用。 回想一下3.6.3节， 一个 PC 相对地址就是距**程序计数器 (PC)** 的当前运行时值的偏移量。当 CPU 执行 一条使用 PC 相对寻址的指令时，它就将在指令中编码的 32 位值加上 PC 的当前运 行时值，得到有效地址(如 call 指令的目标)， PC 值通常是下一条指令在内存中的 地址。

* R_X86_64_32

	重定位一个使用 32 位**绝对地址**的引用。通过绝对寻址， CPU 直接 使用在指令中编码的 32 位值作为有效地址，不需要进一步修改。



### 7.7.2 重定位符号引用

#### Practice Problem 7.5

考虑目标文件 m.o 中对 swap 函数的调用

```assembly
9: e8 00 00 00 00 callq e <main+0xe> ;swap()
```

它的重定位条目如下:

```c
r.offset = 0xa
r.symbol = swap
r.type = R_X86_64_PC32
r.addend = -4
```

现在假设链接器将 m.o 中的 .text 重定位到地址 0x4004d0, 将 swap 重定位到地址0x4004e8

那么 callq 指令中对 swap 的重定位引用的值是什么？
$$
refaddr=0x4004d0+0xa=0x4004da \newline
*refptr= 0x4004e8-4-0x4004da=a
$$


## 7.8 可执行目标文件

一个典型的ELF可执行文件的结构如下：

{% asset_img Snipaste_2021-12-02_18-32-01.png %}

segment header table 描述了程序中连续的 segment 到 连续内存段段映射关系



## 7.9 加载可执行目标文件

加载器（loader）将可执行目标文件中的代码和数据从磁盘复制到内存中，然后通过跳转到程序的**第一条指令**或**入口点**来运行该程序。这个将程序复制到内存并运行的过程叫做加载。

{% asset_img Snipaste_2021-12-02_18-46-37.png %}



## 7.10 动态链接共享库 .so

共享库 shared library 是一个目标模块，在运行或加载时，可以加载到任意的内存地址，并和一个在内存中的程序链接起来。这个过程就是**动态链接** dynamic linking

共享库是被所有引用它的程序**共享**的，而不是被复制到每一个程序内部。这也就是说，系统中只会有共享库的一个文件，它不会被复制。

动态链接的基本思路是：当创建可执行文件时，静态执行一些链接，然后在程序加载时，动态完成链接过程

如下图所示：

{% asset_img Snipaste_2021-12-02_19-41-33.png %}

当加载器加载和运行可执行文件 prog21 时（图中的可执行文件），它注意到 prog21 包含一个 .interp 节，这一节包含动态 链接器的路径名，动态链接器本身就是一个共享目标 （在linux中是 ld-linux.so）。然后链接器将控制权交给这个动态链接器，动态链接器执行下面的重定位完成链接任务：

* 重定位 libc.so 的文本和数据到某个内存段
* 重定位 libvector.so 的文本和数据到另一个内存段
* 重定位 prog21 中所有对由 libc.so 和 libvector.so 定义的符号的引用

最后，动态链接器将控制传递给应用程序。从这个时刻开始，共享库的位置就固定了，并且在程序执行的过程中都不会改变 



## 7.11 从应用程序中加载和链接共享库

Linux 系统为动态链接器提供了一个简单的接口，允许应用程序在**运行时**加载和链接**共享库**

```c
#include <dlfcn.h>
void *dlopen(const char *filename, int flag);
//Returns: pointer to handle if OK, NULL on error
```



```c
#include <dlfcn.h>
void *dlsym(void *handle, char *symbol);
//Returns: pointer to symbol if OK, NULL on error
```

dlsym 函数的输入是一个指向前面已经打开了的共享库的句柄和一个 symbol 名字，如果该符号存在，就返回符号的地址，否则返回 NULL

```c
#include <dlfcn.h>
int dlclose (void *handle);
//Returns: 0 if OK, −1 on error
```

如果没有其他共享库还在使用这个共享库, dlclose函数就卸载该共享库

```c
#include <dlfcn.h>
const char *dlerror(void);
//Returns: error message if previous call to dlopen, dlsym, or dlclose failed;
//NULL if previous call was OK
```

dlerror 函数返回一个字符串，它描述的是调用 dlopen、 dlsym 或者 dlclose 函数时发生的最近的错误，如果没有错误发生，就返回 NULL

这是一个使用上面介绍的函数的例子：

```c
#include <stdio.h>
#include <stdlib.h>
#include <dlfcn.h>

int x[2] = {1, 2};
int y[2] = {3, 4};
int z[2];

int main(){
    void *handle;
    void (*addvec)(int *, int *, int *, int);
    char *error;

    /* Dynamically load the shared library containing addvec() */
    handle = dlopen("./libvector.so", RTLD_LAZY);
    if (!handle) {
        fprintf(stderr, "%s\n", dlerror());
        exit(1);
    }    

    /* Get a pointer to the addvec() function we just loaded */
    addvec = dlsym(handle, "addvec");
    if ((error = dlerror()) != NULL) {
        fprintf(stderr, "%s\n", error);
        exit(1);
    }

    /* Now we can call addvec() just like any other function */
    addvec(x, y, z, 2);
    printf("z = [%d %d]\n", z[0], z[1]);

    /* Unload the shared library */
    if (dlclose(handle) < 0) {
        fprintf(stderr, "%s\n", dlerror());
        exit(1);
    }
    return 0;
}    
```

用以下命令编译这个程序

```shell
linux> gcc -rdynamic -o prog2r dll.c -ldl
```



## 7.12 位置无关代码

可以加载而无需重定位的代码称为位置无关代码 (Position-Independent Code, PIC) 。 用户对 GCC 使用 ***-fpic*** 选项指示 GNU 编译系统生成 PIC 代码。 共享库的编译**必须总是** 使用该选项 



## 7.13 库打桩机制

Linux 链接器支持一个很强大的技术，称为库打桩 (library interpositioning), 它允许 你截获对共享库函数的调用，取而代之执行自己的代码。使用打桩机制，你可以追踪对某 个特殊库函数的调用次数，验证和追踪它的输入和输出值，或者甚至把它替换成一个完全 不同的实现。

在编译、链接和运行时都可以进行库打桩



## 7.14 处理目标文件的工具

GNU binutils 包

* AR: 创建**静态库**，插入、删除、列出和提取成员 。

- STRINGS: 列出一个目标文件中所有可打印的字符串。
- STRIP: 从 目标文件 中删除符号表信息 。
- NM: 列出一个目标文件的符号表中定义的符号。
- SIZE: 列出目标文件中节的名字和大小 。
- READELF: 显示 一 个目标文件的完整结构，包括 ELF 头中编码的所有信息。包含SIZE 和 NM 的 功能。
- OBJDUMP: 所有二进制 工具之母。能够显示一个目标文件中所有的信息。它最大的作用是**反汇编 .text 节中的二进制指令 **。

Linux 系统为操作共享库还提供了 LDD 程序:

* LDD: 列出一个可执行文件在运行时所需要的共享库 。
