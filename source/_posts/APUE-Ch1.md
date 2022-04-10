---
title: UNIX环境高级编程 第一章 -- 学习笔记
categories:
  - notes
date: 2021-07-30 19:27:55
tags:
---


{% asset_img 1.jpg %}

 大名鼎鼎的APUE 第一章 UNIX基础知识

<!-- more -->

# 第一章 UNIX基础知识

## 1.1 引言

这本书阐述了不同版本的 UNIX 操作系统所**提供的服务**



## 1.2 UNIX 体系结构

{% asset_img Snipaste_2021-07-28_11-50-37.png %}

上图就是UNIX操作系统的架构。内核（kernel）位于系统的最内层，外部只能通过系统调用（system calls）与内核通信。

在系统调用外部是公用函数库（library routines）和 shell ，应用程序既可以使用公用函数库，也可以使用系统调用。 shell 是一个特殊的**应用程序**，为运行其他应用程序提供了一个接口。



## 1.3 登录

### 1.登录名

登录名和用户相关信息存储在 `/etc/passwd` 文件中，例如我的账户 `ming` 在该文件中的记录是：

```shell
ming:x:1003:1000::/home/ming:/bin/bash
```

该记录由七个被冒号分隔开的字段组成，依次是 用户名（ming）、加密口令、账户ID（1003）、所在组ID（1000）、注释字段（空白）、起始目录（/home/ming/）和使用的 shell 程序（/bin/bash）



## 1.4 文件和目录



### 1.文件系统

UNIX文件系统是目录和文件的一种层次结构，所有东西的起点都是 **根（root）**这个目录，符号是 `/`

**目录（directory）**是一种**文件**，它包含了**目录项**，目录项又包含一个文件名和文件的属性信息，那些属性信息说明了该目录项是文件还是目录、文件的大小及所有者等等。`stat` 函数可以查看文件的信息结构。



### 3.路径名

中文版第四页实现了一个简单的 ls 命令 

[简单的 ls 命令](https://github.com/imzhangjinming/APUE/blob/master/1/myls.c)

其中引用了 `dirent.h` 头文件，在我的电脑上，它位于 `/usr/include/dirent.h`  ，里面定义了打开文件夹，读取文件夹内容需要使用的一些结构体和函数，如 `DIR` `struct dirent` `opendir()` `readdir()`  `closedir()` 等等



## 1.5 输入和输出

### 1. 文件描述符 file descriptor

通常是一个小的非负整数，内核用它来标识一个特定进程正在访问的文件。在读写文件的时候，可以使用这个描述符

### 2. 标准输入 、标准输出和标准错误

standard input，standard output， standard error

这三个都可以使用 `>` `<`	重定向

### 3.不带缓冲的 I/O

函数 `open` `read` `write` `lseek` `close` 提供了**不带缓冲**的 I/O

[将标准输入复制到标准输出的程序](https://github.com/imzhangjinming/APUE/blob/master/1/copy_stdin2stdout.c)



## 1.6 程序和进程

### 1.程序 program

是一个存储在磁盘上的可执行文件。内核使用 `exec` 函数将其读入内存并执行。

### 2. 进程和进程ID

UNIX系统确保每一个进程都有一个唯一的数字标识，称为进程ID process ID

### 3. 进程控制

主要函数 `fork` `exec` `waitpid`



## 1.7 出错处理

UNIX系统函数出错时通常会返回一个**负值**， `errno` 通常被设置为特定的能够反应为何出错的值。

使用 `man 3 errno` 查看可以赋值给 `errno` 的值

C标准定义了两个函数来打印出错信息：

```c
#include<string.h>
char *strerror(int errnum);
```

`strerror(int errnum)` 返回 `errnum` 对应的出错消息的字符串，比如 permission denied

```c
#include<stdio.h>
void perror(const char *msg);
```

`perror`  首先打印 `msg` 然后接一个 冒号，然后打印现在 `errno` 对应的字符消息

[使用上述两个函数的示例](https://github.com/imzhangjinming/APUE/blob/master/1/use_strerror_perror.c)



* 出错处理

  出错分为两类：致命性的和非致命性的。对于致命性错误，**无法执行恢复动作**。对于非致命性的错误，典型的恢复操作是延迟一段时间，然后重试。

## 1.8 用户标识



### 2. 组ID group ID

组文件 `/etc/group` 负责把组名（字符串）映射成组ID（数字）

`getuid()` `getgid()`  两个函数分别获得现在的用户ID和组ID



## 1.9 信号

信号是用来向进程传递信息的，告诉它发生了什么。进程接收到信号后有以下三种处理方式：

* 什么也不干
* 按默认方式处理。比如对于除数为0这种信号，默认行为是终止进程。
* 自己提供一个处理函数，对应的信号发生时就调用这个函数进行处理



## 1.10 时间值

UNIX有两种时间值：

* 日历时间，记录从1970年1月1日 00:00:00 至今经历的秒数，类型为 size_t
* 进程时间，度量程序运行的快慢。类型为 clock_t

 `time` 函数可以用来查看程序运行时间



## 1.11 系统调用 system call 和库函数 

`man 2 xxx`  可以查看系统调用

`man 3 xxx`  可以查看库函数

