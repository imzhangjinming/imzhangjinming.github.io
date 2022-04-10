---
title: UNIX环境高级编程 第十三章 守护进程
date: 2021-08-11 16:27:49
categories:
- notes
tags:
---


# 第十三章 守护进程 daemon process

## 13.1 引言

这是生存周期长的一种进程，**没有**控制终端，在后台运行。、

多用来处理日常事务活动

<!-- more -->

## 13.2 守护进程的特征

ps 命令打印系统中各个进程的状态



## 13.3 编程规则

在编写守护进程程序时需要遵循一些基本规则，以防止产生不必要的交互作用：

* 首先调用 `umask` 将文件模式创建屏蔽字设置为一个已知值（通常是0）
* 调用 `fork` ，然后使父进程 `exit` 。这样做的目的有两个：1. 如果此进程以一条 shell 命令启动，这样做会让 shell 认为此条命令已经执行完毕； 2. 子进程虽然继承了父进程的进程组ID，但是它有一个新的进程ID，这就保证了子进程不是一个进程组的组长进程，这是调用 `setsid` 的先决条件
* 调用 `setsid` 创建一个新会话。由于子进程不是进程组组长进程，根据9.5节交代的，具体会发生三件事
  * 该子进程变成新会话的领导，此时此刻，该进程是会话中的唯一进程
  * 该子进程成为一个新进程组的组长，进程组ID就是该子进程的进程ID
  * 该进程没有控制终端

* 将当前工作目录更改为**根目录**
* 关闭不再需要的文件描述符



[初始化守护进程](https://github.com/imzhangjinming/APUE/blob/master/13/daemonize.c)

## 13.4 出错记录

广泛使用的出错记录设施是 syslog ，它的组织结构如下图所示

{% asset_img Snipaste_2021-08-11_09-39-50.png %}

* 内核可以调用 `log` 函数向 /dev/klog 写入记录
* 通过 UDP 协议发送到 514 端口
* 大多数用户进程调用 `syslog` 函数 将记录写入 /dev/log 

`syslogd` 守护进程读取以上3种日志消息。此进程在启动的时候读取一个配置文件，通常名为 /etc/syslog.conf ，该文件决定了不同种类的消息应该送往何处



```C
#include <syslog.h>

void openlog(const char *ident, int option, int facility);

void syslog(int priority, const char *format, ...);

void closelog(void);

int setlogmask(int maskpri);
//Returns: previous log priority mask value
```

`openlog` 的 `option` 参数指定各种选项的位屏蔽，它有以下选项：

{% asset_img Snipaste_2021-08-11_09-57-16.png %}

`openlog` 的 `facility` 参数可以说明来自不同设施的消息将以不同的方式处理，它的可选值如下

{% asset_img Snipaste_2021-08-11_10-02-12.png %}

`syslog` 的 priority 参数是 facility 和 level 的组合

{% asset_img Snipaste_2021-08-11_10-10-49.png %}

`format`  参数以及其他所有参数传至 `vsprintf` 函数以便进行格式化。`format` 参数中的每个 `%m` 字符 会被替换成 `errno` 值对应的字符串



`setlogmask` 函数设置进程的记录优先级



## 13.5 单例守护进程 single-instance daemons

单例 指的是在任何时刻**只运行一个**实例

对文件使用**记录锁**为单例守护进程的实现提供了一种简便的方法



[使用文件和记录锁保证只运行一个守护进程的一个副本](https://github.com/imzhangjinming/APUE/blob/master/13/single_instance_daemon.c)



## 13.6 守护进程的惯例

* 若守护进程使用**锁文件**，那么该文件通常存储在 /var/run 目录中并命名为 *name*.pid  。其中 *name* 是守护进程或服务的名字。例如 `cron` 守护进程锁文件的名字 /var/run/crond.pid
* 如果守护进程支持**配置选项**，那么配置文件通常存放在 /etc 目录中，通常命名为 *name*.conf 。其中 *name* 是守护进程或者服务的名字。例如 `syslogd` 的配置文件名字 /etc/syslog.conf
* 守护进程可以用命令行启动，但它们通常是由系统初始化脚本启动的
* 如果一个守护进程有一个配置文件，那么当该守护进程启动时就会读取该文件，但在此之后一般就不会再查看它。



## 13.7 客户-服务器进程模型

守护进程通常作为服务器进程。

