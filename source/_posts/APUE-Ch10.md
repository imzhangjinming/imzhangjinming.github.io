---
title: UNIX环境高级编程 第十章 信号
date: 2021-08-09 15:47:21
categories:
- notes
tags:
---

{% asset_img electrocardiogram-3719883_1920.jpg %}

APUE 第十章 信号



<!-- more -->

# 第十章 信号

## 10.2 信号概念

首先，所有信号都以 `SIG	`  开头。在头文件 `<signal.h>` 中，信号名都被定义为正整数常量，不存在编号为0的信号。

信号是**异步事件**的经典例子。产生信号的事件对进程而言是**随机出现**的。

在某个信号出现时，可以告诉内核按以下3种方式之一进行处理：

* 忽略此信号，什么也不干。
* 捕捉信号。通知内核在某种信号发生时调用一个用户函数。在用户函数中对此信号进行处理。
* 执行系统默认动作。



下图说明了所有信号的名字，哪些系统支持此信号以及对于这些信号，系统的默认动作。

{% asset_img p350.png %}



## 10.3 signal 函数

```C
#include <signal.h>

void (*signal(int signo, void (*func)(int)))(int);
//Returns: previous disposition of signal (see following) if OK, SIG_ERR on error
```

首先，`signal` 是一个函数，它的参数是 `int` 和 函数指针，它的返回值是一个函数指针。参数的函数指针和返回值的函数指针都指向 参数为 `int` ，没有返回值的函数

`func` 参数或者是 `SIG_IGN` 或者是 `SIG_DFL` 或者是 当接收到此信号后要**调用的函数的地址**

[捕捉信号的例子](https://github.com/imzhangjinming/APUE/blob/master/10/catch_signal.c)



## 10.5 中断的系统调用

早期的UNIX系统的一个特性是：如果进程在一个**低速系统调用**内阻塞了，并且在这期间捕捉到一个信号，则中断该系统调用不再执行。`errno` 被设置为 `EINTER`

这样做的原因是基于这样一个假设，进程在阻塞期间捕捉到信号，有很大可能是发生了想要唤醒进程的事情。

低速系统调用是可能会使进程**永远阻塞**的一类**系统调用**：

* 如果某些类型文件的数据不存在，则**读操作**可能会使调用者永远阻塞
* 如果写入的数据不能被文件直接接受，则**写操作**可能会永远阻塞调用者
* 打开特定文件类型可能会永远阻塞调用者，因为要等待特定的条件发生（例如要打开一个终端设备，需要先等待与之连接的调制解调器应答）
* `pause` 和 `wait` 函数
* 某些 `ioctl` 操作
* 某些 **进程间通信** 函数



## 10.6 可重入函数

进程捕捉到信号并对其进行处理时，首先执行该信号处理程序中的指令。如果信号处理程序返回，则继续执行在捕捉到信号时正在执行的正常指令序列。但是信号处理程序可能会对正常指令序列的工作造成破坏。

为此，SUS说明了在信号处理程序中保证调用安全的函数。这些函数被称为**可重入函数**，并被成为是**异步信号安全的（async-signal safe）**。下图列出了异步信号安全的函数

{% asset_img Snipaste_2021-08-08_10-25-51.png %}

注意，在信号处理函数中调用不可重入函数会产生不可预知的结果，看下面的示例

[在信号处理函数中调用不可再入函数](https://github.com/imzhangjinming/APUE/blob/master/10/call_not_reentrant.c)



## 10.8 可靠信号术语和语义

首先需要定义一些在讨论信号过程中需要用到的术语。

当能够引发信号的事件发生时，我们称为进程**产生（generated）**了一个信号

当针对该信号采取行动时，我们称信号已被**递送到（delivered to）**进程。

在 产生 和 递送到 这两个时间点之间的信号状态称为 **未决的（pending）**



进程可以选择一个 “阻塞信号递送”，如果：进程选择了阻塞信号递送，且对该信号的动作是 **默认** 或 **捕捉**，则将该信号保持为 **未决状态** 直到解除该信号的阻塞



每个进程都有一个 **信号屏蔽字（signal mask）** ，它定义了进程想要阻塞的信号的集合。



## 10.9 kill 和 raise 函数

`kill` 函数将信号发送到进程或进程组；`raise` 函数允许 进程向自身发送信号

```C
#include <signal.h>

int kill(pid_t pid, int signo);

int raise(int signo);
//Both return: 0 if OK, −1 on error
```

`kill` 的参数 `pid` 有四种情况：

* `pid>0` 将信号发送到 `pid` 进程
* `pid==0` 将信号发送到与发送进程**属于同一进程组的所有进程**
* `pid<0` 将信号发送到进程组ID等于 `pid` 绝对值 的所有进程
* `pid==-1` 将该信号发送到发送进程有权限向它们发送信号的所有进程



## 10.10 alarm 和 pause 函数

`alarm` 函数设置一个定时器，时间到了之后会产生 `SIGALRM` 信号，默认动作是终止调用 `alarm` 的进程

```C
#include <unistd.h>

unsigned int alarm(unsigned int seconds);
//Returns: 0 or number of seconds until previously set alarm
```



每个进程只能有一个闹钟时间。如果调用  `alarm` 的时候，前一个闹钟还没有超时，则将上一个闹钟的剩余时间作为此次调用的返回值返回，用此次闹钟替换之前的闹钟。

 `alarm(0)` 可以关闭未终止的闹钟



`pause` 函数使调用进程挂起直至捕捉到一个信号

```C
#include <unistd.h>

int pause(void);
//Returns: −1 with errno set to EINTR
```



涉及信号的程序需要有精细而周到的考虑，下面的函数简单实现了 sleep 函数。但是通过测试函数，我们发现，这个实现仍然存在提前结束其他信号处理函数的问题。

[sleep2 函数](https://github.com/imzhangjinming/APUE/blob/master/10/sleep2.c)

[sleep2 测试函数](https://github.com/imzhangjinming/APUE/blob/master/10/test_sleep2.c)



 ## 10.11 信号集

信号集是一种能表示多个信号的**数据类型**—— sigset_t

```C
#include <signal.h>

int sigemptyset(sigset_t *set);

int sigfillset(sigset_t *set);

int sigaddset(sigset_t *set, int signo);

int sigdelset(sigset_t *set, int signo);
//All four return: 0 if OK, −1 on error

int sigismember(const sigset_t *set, int signo);
//Returns: 1 if true, 0 if false, −1 on error
```

`sigemptyset`  初始化由 set 指向的信号集，**清除**其中所有信号

`sigfillset`  初始化由 set 指向的信号集，使其**包含**所有信号

`sigaddset`  将一个信号添加到已有的信号集种

`sigdelset` 将一个信号从指定的信号集删除



## 10.12 sigprocmask 函数

使用这个函数可以检测或更改一个进程的信号屏蔽字。信号屏蔽字规定了当前组测不能递送给该进程的信号集

```C
#include <signal.h>

int sigprocmask(int how, const sigset_t *restrict set,sigset_t *restrict oset);
//Returns: 0 if OK, −1 on error
```

首先，如果 `oset` 是一个非空指针，那么进程当前的信号屏蔽字通过 `oset` 返回

其次，如果 `set` 是一个非空指针，则 `how` 参数指示如何修改当前信号屏蔽字

{% asset_img Snipaste_2021-08-08_14-23-08.png %}

如果 `set` 是一个空指针，则不改变进程的信号屏蔽字， `how` 值也就没有意义



## 10.13 sigpending 函数

```C
#include <signal.h>

int sigpending(sigset_t *set);
//Returns: 0 if OK, −1 on error
```

`sigpending` 函数通过 `set` 参数返回调用进程当前被阻塞不能递送和未决的信号集

[一个使用了前面介绍的很多函数的例子](https://github.com/imzhangjinming/APUE/blob/master/10/use_sigprocmask.c)



## 10.14 sigaction 函数

此函数可以检查并修改与指定信号相关联的处理动作

```C
#include <signal.h>

int sigaction(int signo, const struct sigaction *restrict act,struct sigaction *restrict oact);
//Returns: 0 if OK, −1 on error
```

类似地， `act` 指明想要设置的动作， `oact` 返回该信号的上一个动作，`signo` 指明想要设置的信号

结构体 `sigaction` 内容如下

```C
struct sigaction {
	void	(*sa_handler)(int);	/* addr of signal handler, */
								/* or SIG_IGN, or SIG_DFL */
	sigset_t sa_mask;			/* additional signals to block */
	int	sa_flags;				/* signal options, Figure 10.16 */
    
	/* alternate handler */
	void (*sa_sigaction)(int, siginfo_t *, void *);
};
```

下图说明了 `sigaction` 结构体中 `sa_flags` 可选项

{% asset_img p385.png %}

[用 sigaction 实现 signal 函数](https://github.com/imzhangjinming/APUE/blob/master/10/mysignal.c)



## 10.15 sigsetjmp 和 siglongjmp 函数

在信号处理程序中进行 **非局部转移** 时应当使用的函数

```C
#include <setjmp.h>

int sigsetjmp(sigjmp_buf env, int savemask);
//Returns: 0 if called directly, nonzero if returning from a call to siglongjmp

void siglongjmp(sigjmp_buf env, int val);
```

这两个函数和 `setjmp` 、 `longjmp` 之间的唯一区别是 `sigsetjmp` 增加了一个参数。如果 `savemask` 非 0 ，则`sigsetjmp` 在 `env` 中保存当前信号屏蔽字，这样在 调用 `siglongjmp` 时会自动恢复信号屏蔽字。

[使用这两个函数的例子](https://github.com/imzhangjinming/APUE/blob/master/10/sigsetjmp_siglongjmp.c)



## 10.16 sigsuspend 函数

```C
#include <signal.h>

int sigsuspend(const sigset_t *sigmask);
//Returns: −1 with errno set to EINTR
```

此函数用来在一个原子操作中将进程的屏蔽字设置为 `sigmask` 指向的对象并使进程休眠。如果一个**信号被捕捉**到并且信号**处理函数返回**，那么 `sigsuspend` 返回，并且信号**屏蔽字**被设置为调用 `sigsuspend` **之前的状态**。

此函数的应用：

* 首先改变进程的屏蔽字，使接下来的代码不会被某种信号中断。等需要保护的代码执行完毕后，调用 `sigsuspend` 函数恢复进程的屏蔽字并等待信号到来



可以使用信号实现 第八章中介绍到的 `TELL_WAIT`, `TELL_PARENT`, `TELL_CHILD`, `WAIT_PARENT`, and `WAIT_CHILD`

[以上 5 个函数的一种实现](https://github.com/imzhangjinming/APUE/blob/master/10/sync_parent_child.c)



## 10.17 abort 函数

此函数功能是使程序**异常**终止

```C
#include <stdlib.h>

void abort(void);
//This function never returns
```

此函数将 SIGABRT 信号发送给调用进程。而且 ISO C 规定，即使该信号被捕捉到，并在相应的信号处理函数中返回，程序也不会回到调用它的函数内。

因为让进程捕捉 SIGABRT 信号的意图 是：在进程终止之前做一些必要的清理工作。

[abort 的 POSIX.1 实现 ](https://github.com/imzhangjinming/APUE/blob/master/10/POSIX.1_abort.c)



## 10.18 system 函数

在 8.13 节曾实现过一个简单的 `system` 函数，那个函数没有进行信号处理，而POSIX.1 要求 `system` 忽略 `SIGINT` 和 `SIGQUIT` ，阻塞 `SIGCHLD`。

[带信号处理的 system 函数实现](https://github.com/imzhangjinming/APUE/blob/master/10/system_with_signal.c)



## 10.19 sleep 、 nanosleep 和 clock_nanosleep 函数

```C
#include <unistd.h>

unsigned int sleep(unsigned int seconds);
//Returns: 0 or number of unslept seconds
```

此函数使调用进程挂起直至满足下面两个条件之一

* 已经过了 `seconds` 所指定的时间
* 调用进程捕捉到一个信号并从信号处理程序返回

第一种情况下，sleep 返回 0 ；第二种情况下，返回未休眠完的秒数（要求的秒数-实际休眠的秒数）

[sleep 函数的一种可靠实现](https://github.com/imzhangjinming/APUE/blob/master/10/reliable_sleep.c)



`nanosleep` 提供了纳米级的休眠精度

```C
#include <time.h>

int nanosleep(const struct timespec *reqtp, struct timespec *remtp);
//Returns: 0 if slept for requested time or −1 on error
```

`clock_nanosleep` 函数提供了相对于**特定时钟**的延迟时间来挂起调用线程

```C
#include <time.h>

int clock_nanosleep(clockid_t clock_id, int flags,const struct timespec *reqtp, struct timespec *remtp);
//Returns: 0 if slept for requested time or error number on failure
```



## 10.20 sigqueue 函数

使用排队信号必须做以下几个操作

* 使用 `sigaction` 函数指定信号处理程序时设置 `SA_SIGINFO` 标志
* 在 `sigaction` 结构的 `sa_sigaction` 成员中提供信号处理程序
* 使用 `sigqueue` 函数发送信号

```C
#include <signal.h>

int sigqueue(pid_t pid, int signo, const union sigval value)
//Returns: 0 if OK, −1 on error
```

此函数只能将信号发送到**单个进程**。除了有一个 `value` 参数可以传递额外的信息外，此函数与 `kill` 函数类似

信号**不能无限**排队。最大排队数量受 `SIGQUEUE_MAX` 限制



## 10.22 信号名和编号

可以使用 psignal 函数打印与信号编号对应的字符串

```C
#include <signal.h>

void psignal(int signo, const char *msg);
```

打印的方式是： “msg： signo代表的信号的说明”

如果 `msg` 为 `NULL`，就只打印信号的说明

```C
#include <signal.h>

void psiginfo(const siginfo_t *info, const char *msg);
```



下面这个函数直接返回给定信号对应的字符串

```C
#include <string.h>

char *strsignal(int signo);
//Returns: a pointer to a string describing the signal
```



# 课后练习

## 2 实现10.22节中说明的 sig2str 函数

[我的解答](https://github.com/imzhangjinming/APUE/blob/master/10/exercise/mysig2str.c)

## 6 编写一段程序测试 [这些函数] 。要求进程创建一个文件并向文件写一个整数0，然后，进程调用 fork ，接着，父进程和子进程交替增加文件中的计数器值，每次计数器值增加1时，打印是哪一个进程进行了该增加1的操作

[我的解答](https://github.com/imzhangjinming/APUE/blob/master/10/exercise/test_sync_p_c.c)

