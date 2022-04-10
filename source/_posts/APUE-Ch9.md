---
title: UNIX环境高级编程 第九章 进程关系
date: 2021-08-07 23:54:58
categories:
- notes
tags:
---
暂时觉得第九章的东西不是那么重要

<!-- more -->

# 第九章 进程关系


## 9.4 进程组

每个进程除了有一个进程ID之外，还属于一个**进程组**。同一进程组中的各个进程接收来自同一终端的各种**信号**，并且有唯一的**进程组ID**，使用 `getpgrp` 返回调用进程的进程组ID

```C
#include <unistd.h>

pid_t getpgrp(void);
//Returns: process group ID of calling process
```



下面这个 `getpgid` 函数可以获得指定进程所在进程组ID

```C
#include <unistd.h>

pid_t getpgid(pid_t pid);
//Returns: process group ID if OK, −1 on error
```



每个进程组有一个**组长进程**，进程组ID就是组长进程ID。进程组组长可以**创建一个进程组**、创建该组中的进程，然后终止。只要在某个进程组中有一个进程存在，该进程组就存在，这与其组长进程是否终止**无关**。

进程调用 `setpgid` 可以加入一个现有的进程组，或者创建一个新的进程组

```C
#include <unistd.h>

int setpgid(pid_t pid, pid_t pgid);
//Returns: 0 if OK, −1 on error
```

这个函数将 进程 `pid` 的进程组ID设置为 `pgid` 。如果

* `pid=pgid` ，以 `pid` 进程为进程组组长创建进程组
* `pid=0`，则使用调用进程的进程ID，或者加入进程组，或者创建进程组
* `pgid=0`，则使用 `pid` 进程ID作为进程组ID



一个进程**只能**设置它自己和它子进程的进程组ID。如果一个子进程执行了7个 **exec** 函数之一，那么其父进程就不能再更改它的进程组ID了



## 9.5 会话 session

这是一个或多个**进程组的集合**。下图展示了一个包含三个进程组的会话

{% asset_img Snipaste_2021-08-07_17-21-05.png %}

进程可以调用 `setsid` 函数建立一个新会话

```C
#include <unistd.h>

pid_t setsid(void);
//Returns: process group ID if OK, −1 on error
```

根据调用进程的不同，分为两种情况

* 调用进程不是所在进程组的组长进程，则此函数**创建一个新会话**。具体发生以下3件事
  * 该进程变成新会话的领导（session leader），此时此刻，该进程是会话中的唯一进程（光杆司令）
  * 该进程成为一个新进程组的组长进程。新进程组ID就是该进程ID
  * 该进程没有控制终端。

* 调用进程已经是一个进程组的组长进程，则此函数出错返回



`getsid` 函数返回会话首进程的进程组ID（也就是会话首进程ID）

```C
#include <unistd.h>

pid_t getsid(pid_t pid);
//Returns: session leader’s process group ID if OK, −1 on error
```

如果 `pid=0` ，此函数返回**调用进程**所在的会话的会话首进程的进程组ID。如果 `pid` 不属于**调用者**所在的会话，那么调用进程就**不能得到**会话首进程的进程组ID



## 9.6 控制终端 controlling terminal

会话和进程组有下面这些特性

* 一个**会话**可以有一个控制终端。
* 建立与控制终端**连接**的**会话首进程**被称为控制进程（controlling process）
* 一个会话中的几个进程组可以被分为 **一个** 前台进程组（foreground process group）以及 **一个或多个** 后台进程组（background process group）
* 如果一个会话有一个控制终端，那么它只有一个前台进程组，其他进程组都是后台进程组
* 无论何时在终端键入中断键，都会将中断信号发送至**前台进程组的所有进程**
* 无论何时在终端键入退出键，都会将退出信号发送至**前台进程组的所有进程**
* 如果控制终端检测到调制解调器断开连接，则将挂断信号发送至**控制进程**



进程组、会话和控制终端的关系可用下图说明

{% asset_img Snipaste_2021-08-07_18-00-19.png %}



## 9.7 tcgetpgrp 、tcsetpgrp 和 tcgetsid 函数

我们需要一种方法通知内核哪一个进程组是前台进程组，这样终端就能将输入和信号发送到正确的进程组

```C
#include <unistd.h>

pid_t tcgetpgrp(int fd);
//Returns: process group ID of foreground process group if OK, −1 on error

int tcsetpgrp(int fd, pid_t pgrpid);
//Returns: 0 if OK, −1 on error
```

`tcgetpgrp` 函数返回与在 `fd` 上打开的终端相关联的前台进程组ID

`tcsetpgrp` 函数将在 `fd` 上打开的终端的前台进程组ID设置为 `pgrpid` 。当然， `pgrpid` 必须是同一个会话中的一个进程组的ID。 `fd` 必须引用该会话的控制终端



## 9.8 作业控制 job control

作业控制允许在一个终端上启动多个作业（groups of processes）。一个作业只是几个进程的集合，通常是几个进程组成的管道。

作业控制这一小节我基本看不明白，也没有感受到对它的迫切需求，暂时放过它



## 9.10 孤儿进程组 orphan process

孤儿进程是父进程已经终止的进程。同样，一个进程组也可以成为 孤儿



