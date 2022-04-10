---
title: UNIX环境高级编程 第七章 进程环境
date: 2021-08-05 16:29:44
categories:
- notes
tags:
---

{% asset_img network-cables-499792_1920.jpg %}

APUE 第七章 进程环境



<!-- more -->

# 第七章 进程环境

## 7.2 main 函数

C程序总是从 `main` 函数开始执行。

```C
int main(int argc,char **argv);
```

当内核执行C程序时，在调用  `main` 函数前先调用一个特殊的**启动程序**。编译器会调用连接编辑器，后者会将启动程序的地址放置在可执行文件的开始，这样启动程序就会先于  `main` 函数执行，并为  `main` 函数从内核取得**命令行参数**和**环境变量值**，为执行 `main` 函数做好准备。



## 7.3 进程终止

一共有8种方式终止进程，其中5种是正常终止

* 从 `main` 函数返回
* 调用 `exit` 函数
* 调用`_exit` 或 `_Exit` 函数
* 最后一个**线程**从其启动程序返回
* 最后一个**线程**调用 `pthread_exit` 

异常终止有3种

* 调用 `abort` 
* 接到一个信号
* 最后一个**线程** 对**取消请求**做出响应



1. **退出函数**

   ```C
   #include <stdlib.h>
   
   void exit(int status);
   void _Exit(int status);
   
   #include <unistd.h>
   void _exit(int status);
   ```

   `_Exit` 和 `_exit` 直接进入内核，`exit` 函数先执行一些清理操作后再返回内核

   这三个函数都有一个整型参数，称为**退出状态**

   `main` 函数返回一个整型值与使用该整型值调用 `exit` 是等价的 `return 0;` = `exit(0);`

   

2. `atexit` **函数**

   进程可以登记一些函数供 `exit` 自动调用，这些函数被称为 **终止处理程序**（exit handler）。`atexit` 函数就是用来登记这些函数的

   ```C
   #include <stdlib.h>
   
   int atexit(void (*func)(void));
   //Returns: 0 if OK, nonzero on error
   ```

   注意， `exit` 调用这些函数的顺序与它们被登记的顺序**相反**；如果同一个函数多次注册，那么它也会被多次调用

   [使用 `atexit` 函数](https://github.com/imzhangjinming/APUE/blob/master/7/atexit_use.c)



下图显示了一个C程序是如何启动的，以及它终止的各种方式

{% asset_img Snipaste_2021-08-05_08-28-23.png %}





## 7.4 命令行参数

当执行一个程序时，调用 `exec` 的进程可将命令行参数传递给该新程序。



## 7.5 环境表

每个程序都能接收到一张环境表，它是一个字符指针数组，每个指针都指向一个以 null 字符结尾的C字符串（环境字符串）

**环境表的地址**保存在一个被称为**环境指针**的全局变量 `environ` 中

```C
extern char **environ；
```



## 7.6 C程序的存储空间布局

C程序由下列几部分组成：

* Text segment 正文段 。包含机器指令，可共享
* Initialized data segment 初始化了的数据段 。 赋了初值的全局变量
* Uninitialized data segment 未初始化的数据段 ，也称为 **bss**（block started by symbol）段。没赋初值的全局变量
* Stack 栈 。 自动变量及每次函数调用需要保存的信息
* Heap 堆 。 动态内存分配



{% asset_img Snipaste_2021-08-05_08-53-29.png %}



## 7.8 存储空间分配

ISO C 说明了3个用于存储空间动态分配的函数

* malloc 	分配指定字节数的存储区。存储区中的初始值不确定
* calloc      为 指定数量指定长度的对象分配存储空间。存储区中的每一位都初始化为0
* realloc    增加或减少以前分配区的长度。



```C
#include <stdlib.h>

void *malloc(size_t size);

void *calloc(size_t nobj, size_t size);

void *realloc(void *ptr, size_t newsize);
//All three return: non-null pointer if OK, NULL on error

void free(void *ptr);
```



## 7.9 环境变量

环境变量以环境字符串的方式存储：

```shell
name=value
```



可用 `getenv` 函数取得环境变量的值

```C
#include <stdlib.h>

char *getenv(const char *name);
//Returns: pointer to value associated with name, NULL if not found
```

返回的指针指向 `name` 参数对应的 `value` 



可用以下函数修改环境变量的值

```C
#include <stdlib.h>

int putenv(char *str);
//Returns: 0 if OK, nonzero on error

int setenv(const char *name, const char *value, int rewrite);

int unsetenv(const char *name);
//Both return: 0 if OK, −1 on error
```

这三个函数操作如下：

* `putenv` 取形式为 `name=value` 的字符串，将其放到环境表中，如果 `name` 已经存在，则先删除原来的定义
* `setenv` 将 `name` 设置为 `value` 。如果在环境中 `name` 已经存在，那么 1. 如果 `rewrite != 0` ，则首先删除原来的定义； 2. 如果 `rewrite=0`，`name` 不设置为新的 `value` ，程序也不出错

* `unsetenv` 删除 `name` 的定义



## 7.10 setjmp 和 longjmp

这俩函数具有跳转功能，对于处理发生在很深层嵌套函数调用中的出错情况是非常有用的。

```C
#include <setjmp.h>

int setjmp(jmp_buf env);
//Returns: 0 if called directly, nonzero if returning from a call to longjmp

void longjmp(jmp_buf env, int val);
```



1. **自动变量、寄存器变量和易失变量**

   在调用 longjmp 函数后，各种变量会恢复调用 setjmp 处的值吗？

   

   [使用实际程序验证](https://github.com/imzhangjinming/APUE/blob/master/7/longjmp_var.c)

   

   在我的电脑上运行结果：

   不进行任何编译优化的情况下，只有寄存器变量能够恢复首次调用  `setjmp` 时的值；

   使用编译优化之后，自动变量和寄存器变量能够恢复首次调用 `setjmp` 时的值	

   

## 7.11 getrlimit 和 setrlimit 函数

每个**进程**都有一组资源限制，其中一些可以用 getrlimit 和 setrlimit 函数查询和更改

```C
#include <sys/resource.h>

int getrlimit(int resource, struct rlimit *rlptr);

int setrlimit(int resource, const struct rlimit *rlptr);
//Both return: 0 if OK, −1 on error
```



其中 `rlimit` 结构体定义如下

```C
struct rlimit {
	rlim_t rlim_cur; /* soft limit: current limit */
	rlim_t rlim_max; /* hard limit: maximum value for rlim_cur */
};
```



`resource` 可选下列值

{% asset_img Snipaste_2021-08-05_11-19-32.png %}



