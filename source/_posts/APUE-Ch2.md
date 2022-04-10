---
title: UNIX环境高级编程 第二章 -- 学习笔记
categories:
  - notes
date: 2021-07-31 15:11:30
tags:
---


{% asset_img code-1839406_1920.jpg %}

APUE 第二章 UNIX标准及实现

<!-- more -->

# 第二章 UNIX标准及实现



## 2.2 UNIX标准化

### 2.2.1 ISO C

ANSI -- American National Standards Institute

ISO -- International Organization for Standardization

本书中使用的标准是 **C99**



### 2.2.2 IEEE POSIX 

IEEE -- Institute of Electrical and Electronics Engineers

POSIX -- Portable Operating System Interface



### 2.2.3  Single UNIX Specification (SUS)

SUS 是 POSIX.1 标准的一个超集，它定义了一些附加接口扩展了 POSIX.1 规范提供的功能



## 2.3 UNIX 系统实现

### 2.3.1 SVR4

SVR4 (UNIX System V Release 4)是AT&T的UNIX系统实验室（UNIX System Laboratories，USL）的**产品**

### 2.3.2 4.4BSD

BSD -- Berkeley Software Distribution

### 2.3.3 FreeBSD

FreeBSD是在4.4BSD-Lite操作系统上发展而来的

### 2.3.4 Linux

Linus Torvalds 此刻正在我宿舍的墙上看着我。

### 2.3.5 Mac OS X

### 2.3.6 Solaris

Solaris是由Sun Microsystems （现 Oracle）开发的 UNIX 系统版本。基于 SVR4



## 2.5 限制

* 编译时限制，可以在头文件中定义
* 运行时限制，需要进程调用特定的函数获得限制值

但是，有的限制在一个实现中是固定的（头文件中），在另一个实现中是变动的（调用函数确定）。为了解决这类问题，提供以下三种限制：

* 编译时限制（头文件）
* 与文件或目录无关的运行时限制（sysconf）
* 与文件或目录有关的运行时限制（pathconf 和 fpathconf）

总之，限制值要么从**头文件获得**，要么**调用函数获得**



### 2.5.1 ISO C限制

`<limits.h>` 头文件定义了所有编译时限制

{% asset_img Snipaste_2021-07-30_18-02-37.png %}

{% asset_img Snipaste_2021-07-30_18-05-47.png %}

### 2.5.2 POSIX限制

{% asset_img Snipaste_2021-07-30_18-11-14.png %}

图2.8中的有些值并不符合实际，所以具体实现时会有区别。具体实现时通常会将前缀 `_POSIX_` 去掉

{% asset_img Snipaste_2021-07-30_18-17-20.png %}



### 2.5.4 函数 sysconf 、pathconf 、fpathconf

```c
#include<unistd.h>

long sysconf(int name);

long pathconf(const char * pathname,int name);

long fpathconf(int fd,int name);

//以上三个函数，如果运行成功则返回相应的值；不成功则返回-1
```

{% asset_img Snipaste_2021-07-30_18-24-49.png %}

{% asset_img Snipaste_2021-07-30_18-25-11.png %}

`_SC_` 前缀表示是 sysconf 函数的参数

{% asset_img Snipaste_2021-07-30_18-26-18.png %}

`_PC_` 前缀表示是 pathconf 和 fpathconf 函数的参数

关于这三个函数返回值的一些说明：

* 如果 name 参数不合适，三个函数都会返回 -1 ，并把 errno 置为 EINVAL
*  `_SC_CLK_TCK` 的返回值是每秒的时钟滴答数



### 2.5.5 不确定的运行时限制

#### 1.为路径名分配存储空间

[示例程序](https://github.com/imzhangjinming/APUE/blob/master/2/malloc_for_pathname.c)

#### 2. 最大打开文件数

[示例程序](https://github.com/imzhangjinming/APUE/blob/master/2/get_openmax.c)



## 2.6 选项

定义标准的时候会同时定义选项组，也就是可选的接口。不同的实现对可选接口的支持不尽相同，所以，在编写可移植软件的时候就需要一种方法判断实现是否支持某些可选接口。

和上一节对限制的处理方法一样，POSIX.1 定义了3种方法处理选项：

* **编译时选项**定义在 `<unistd.h>` 中
* 用 `sysconf`  函数来判断与文件或目录无关的**运行时选项**
* 用 `pathconf` 和 `fpathconf`  函数来判断与文件或目录有关的**运行时选项**

对于每一个选项，有以下三种支持状态：

* 符号常量没有定义或者定义为 -1 ，平台不支持那个选项
* 符号常量定义值  >0  , 平台支持那个选项
* 符号常量定义值  =0  , 使用 `sysconf` `pathconf` `fpathconf`  函数来判断



{% asset_img Snipaste_2021-07-31_09-37-36.png %}

{% asset_img 1.png %}

## 2.7 功能测试宏

在各种头文件中，很多实现添加了自己定义的符号，那么如何在编译的时候只使用 POSIX.1 和 XSI 定义的符号呢？

方法是定义常量 `_POSIX_C_SOURCE`  和  `_XOPEN_SOURCE`  



## 2.8 基本系统数据类型

头文件 `<sys/types.h>` 中定 义了与实现有关的数据类型，称为 基本系统数据类型 (primitive system data type) ，通常以 `_t`  结尾

{% asset_img Snipaste_2021-07-31_09-51-17.png %}

