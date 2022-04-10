---
title: UNIX环境高级编程 第八章 进程控制
date: 2021-08-07 16:23:44
categories:
- notes
tags:
---

{% asset_img turntable-1337986_1920.jpg %}
UNIX环境高级编程 第八章 进程控制


<!-- more -->

# 第八章 进程控制

## 8.2 进程标识

每一个进程都有一个非负整型表示的**唯一**进程ID。

当然除了进程ID这个标识符之外，每个进程还有一些其他标识符。下列函数返回这些标识符

```C
#include <unistd.h>

pid_t getpid(void);
//Returns: process ID of calling process

pid_t getppid(void);
//Returns: parent process ID of calling process

uid_t getuid(void);
//Returns: real user ID of calling process

uid_t geteuid(void);
//Returns: effective user ID of calling process

gid_t getgid(void);
//Returns: real group ID of calling process

gid_t getegid(void);
//Returns: effective group ID of calling process
```



##  8.3 fork 函数

一个现有的进程可以调用 `fork` 函数创建一个**新进程**

```C
#include <unistd.h>

pid_t fork(void);
//Returns: 0 in child, process ID of child in parent, −1 on error
```

 `fork` 函数被调用一次，但是返回两次。两次返回的**区别**是子进程的返回值是 0，而父进程的返回值则是新建子进程的进程ID。

子进程是父进程的**副本**

[使用 fork 函数](https://github.com/imzhangjinming/APUE/blob/master/8/fork_use.c)



一般来说，在 `fork` 之后是父进程先执行还是子进程先执行是不确定的，取决于内核。



**文件共享**

 `fork` 函数的一个特性是父进程的所有打开文件**描述符**都被复制到子进程当中。父进程与子进程每个相同的打开描述符共享一个**文件表项**，这就说明了重要的一点，父进程和子进程共享同一个**文件偏移量**。

下图说明了 `fork` 之后父进程与子进程之间的文件共享关系

{% asset_img Snipaste_2021-08-05_17-40-56.png %}



在  `fork` 之后处理文件描述符有以下两种常见的情况：

* 父进程等待子进程完成
* 父进程和子进程各自执行不同的程序段。在这种情况下，在 `fork` 之后，父进程和子进程各自关闭他们不需要使用的文件描述符，这样就不会干扰对方使用的文件描述符。这种方法是**网络服务进程**经常使用的



父进程和子进程之间的区别如下：

*  `fork` 的返回值不同
* 进程ID不同
* 各自的父进程ID不同
* 子进程不继承父进程中设置的文件锁
* 子进程的未处理闹钟被清除
* 子进程的未处理信号集设为空集



使用 `fork` 函数**失败**的两个主要**原因**：

* 系统中已经有了太多的进程
* 该实际用户ID的进程总数超过了系统限制



 `fork` 函数有以下两种**用法**：

* 一个父进程希望复制自己，使父进程和子进程同时执行不同的代码段。
* 一个进程要执行一个不同的程序。



## 8.5 exit 函数

在第七章中已经说过，一共有8种方式终止进程，其中5种是正常终止

* 从 `main` 函数返回
* 调用 `exit` 函数
* 调用`_exit` 或 `_Exit` 函数
* 最后一个**线程**从其启动程序返回
* 最后一个**线程**调用 `pthread_exit` 

异常终止有3种

* 调用 `abort` 
* 接到一个信号
* 最后一个**线程** 对 **取消请求** 做出响应



对于以上的任意一种终止情形，我们都希望终止进程能够通知它的父进程它是如何终止的。

* 正常终止时，将 **退出状态**（exit status ）作为 `exit` 或 ` _exit` 或  `_Exit` 的参数传递给函数
* 异常终止时，内核产生一个指示异常终止原因的**终止状态**（termination status） 

在上述两种情况下，父进程都能用 `wait` 或 `waitpid` 函数取得终止进程的终止状态



## 8.6 wait 和 waitpid 函数

首先需要明确的是这俩函数都是父进程调用的。

那么父进程调用这两个函数会发生什么呢？分为以下几种情况：

* 如果其所有子进程都还在运行，则阻塞
* 如果有一个子进程已经终止，正在等待其父进程获取其终止状态，则取得该子进程的终止状态立即返回
* 如果父进程没有任何子进程，则立即出错返回



```C
#include <sys/wait.h>

pid_t wait(int *statloc);

pid_t waitpid(pid_t pid, int *statloc, int options);
//Both return: process ID if OK, 0 (see later), or −1 on error
```

这两个函数区别如下：

* 在一个子进程终止之前，`wait` 使其相对的父进程阻塞；而 `waitpid  ` 有一个选项，可以选择不阻塞父进程
* `waitpid` 并不等待在其调用后的第一个终止子进程，它有若干个选项，可以控制它所等待的进程



`statloc` 参数是一个整型指针，它用来存放终止状态，使用下面的这些宏和终止状态可以判断进程究竟是如何终止的：

{% asset_img Snipaste_2021-08-05_21-07-03.png %}

这些宏包含在 ` <sys/wait.h>` 中



[使用宏打印 exit 信息](https://github.com/imzhangjinming/APUE/blob/master/8/print_exit_status.c)



前面已经说过， 只要任一子进程终止， `wait` 函数就返回；而 `waitpid` 函数不是这样，它有一个 `pid` 参数可以选择它需要等待的进程

* `pid == -1`	等待任一子进程
* `pid > 0`       等待 `pid` 代表的进程
* `pid == 0`     等待组ID等于调用进程组组ID的任一子进程
* `pid < -1`      等待组ID等于`pid` 绝对值的任一子进程

 `waitpid` 函数的另一个参数 `options` 让我们能够进一步控制 `waitpid` 的操作。  

{% asset_img Snipaste_2021-08-05_21-45-48.png %}

[使用两次 fork 来避免僵尸进程](https://github.com/imzhangjinming/APUE/blob/master/8/use_fork_twice.c)



## 8.7 waitid 函数

这是另一个取得进程终止状态的函数，它类似于 `waitpid` 但是比它更灵活

```C
#include <sys/wait.h>

int waitid(idtype_t idtype, id_t id, siginfo_t *infop, int options);
//Returns: 0 if OK, −1 on error
```

`id` 参数用来指定要等待的进程，它的具体含义依赖于 `idtype` 参数：

{% asset_img Snipaste_2021-08-06_09-35-08.png %}



`options` 参数是下面这些标志的按位或运算。这些标志指示调用者关注哪些状态变化：

{% asset_img Snipaste_2021-08-06_09-36-51.png %}

`infop` 参数是指向 `siginfo` 结构体的指针。该结构体包含了造成子进程状态改变有关信号的详细信息



## 8.8 wait3 和 wait4 函数

这俩函数提供的功能比之前介绍的函数多一个，即允许内核返回由终止进程及其所有子进程使用的资源概况

```C
#include <sys/types.h>
#include <sys/wait.h>
#include <sys/time.h>
#include <sys/resource.h>

pid_t wait3(int *statloc, int options, struct rusage *rusage);

pid_t wait4(pid_t pid, int *statloc, int options, struct rusage *rusage);
//Both return: process ID if OK, 0, or −1 on error
```



`rusage` 结构体就是用来保存资源统计信息的，包括用户CPU时间量、系统CPU时间量、缺页次数、接收到信号的次数等



## 8.9 竞争条件 race condition

当多个进程都想对共享数据进行某种处理，而最后的结果又取决于进程运行的顺序时，我们认为发生了竞争条件。

为了合理安排各个进程的执行顺序，需要让进程进行通信以告知对方自己的状态。书中给出了`TELLWAIT` 、            `TELL_PARENT` 、`TELL_CHILD` 、`WAIT_PARENT` 和`WAIT_CHILD` 5个程序，它们可以是函数也可以是宏，有不同的实现方法。

我们先来看一个具有竞争条件的程序：

```C
#include "apue.h"
static void charatatime(char *);

int main(void){
	pid_t	pid;
	if ((pid = fork()) < 0) {
		err_sys("fork error");
	} else if (pid == 0) {
		charatatime("output from child\n");
	} else {
		charatatime("output from parent\n");
	}
	exit(0);
}

static void	charatatime(char *str){
	char	*ptr;
	int		c;
	setbuf(stdout, NULL);
	/* set unbuffered */
	for (ptr = str; (c = *ptr++) != 0; )
		putc(c, stdout);
}
```



[也可以在这里找到这个程序](https://github.com/imzhangjinming/APUE/blob/master/8/race_condition.c)

注意 `charatatime` 函数中将 `stdout` 设置为无缓冲的，于是每个字符输出都需要调用一次 `write` 函数，这样就能使内核尽可能多地在两个进程之间进行切换。



## 8.10 exec 函数

当进程调用一个 exec 函数时，该进程执行的程序完全替换为新的程序，新程序从其 main 函数开始执行。

exec 函数并不创建新进程，所以调用前后进程ID相同，它只是用磁盘上的一个新程序替换了当前进程的正文段、数据段、堆段和栈段。

有7种不同的 exec 函数，它们统称为 exec 函数

```C
#include <unistd.h>

int execl(const char *pathname, const char *arg0, ... /* (char *)0 */ );

int execv(const char *pathname, char *const argv[]);

int execle(const char *pathname, const char *arg0, .../* (char *)0, char *const envp[] */ );

int execve(const char *pathname, char *const argv[], char *const envp[]);

int execlp(const char *filename, const char *arg0, ... /* (char *)0 */ );

int execvp(const char *filename, char *const argv[]);

int fexecve(int fd, char *const argv[], char *const envp[]);
//All seven return: −1 on error, no return on success
```



* 7个函数的第一个区别：

  前四个函数取路径名作为参数，后两个函数取文件名作为参数，最后一个文件取文件描述符作为参数

  以 `filename` 作为参数时，如果它包含 `/` ，则认为它是一个路径，否则就在 `PATH` 路径里寻找可执行文件

* 第二个区别

  l 表示列表 list ，v 表示矢量 vector。函数 `execl` 、`execle` 和 `execlp` 要求将新程序的每个命令行参数都说明为一个单独的参数，这种参数表以**空指针结尾**，像这样

  ```shell
   char *arg0, char *arg1, char *arg2, ...,char *argn, (char *)0
  ```

  另外 4 个函数 `execv` 、`execve` 、`execvp` 和 `fexecve` ，则应先构造一个指向各参数的**指针数组**，然后将该数组地址作为这4个函数的地址

* 第三个区别

  以 e 结尾的三个函数 `execle`  、`execve`  和`fexecve`可以传递一个指向**环境字符串指针数组**的指针。其他四个函数则使用 `environ` 变量为新的程序复制现有的环境



注意，在 `exec` 前后**实际用户ID**和**实际组ID**保持**不变**，而**有效ID**和**有效组ID**是否改变则取决于所执行程序文件的设置用户ID位和设置组ID位是否设置。



这7个函数中只有 execve 是内核的系统调用，另外6个是库函数，它们最终都会调用该系统调用，这7个函数关系如下：

{% asset_img Snipaste_2021-08-06_16-07-41.png %}

[使用 exec 函数](https://github.com/imzhangjinming/APUE/blob/master/8/exec_use.c)



## 8.11 更改用户ID和组ID

可以用 `setuid` 函数设置实际用户ID和有效用户ID，用 `setgid` 函数设置实际组ID和有效组ID

```C
#include <unistd.h>

int setuid(uid_t uid);

int setgid(gid_t gid);
//Both return: 0 if OK, −1 on error
```

关于谁能够更改ID有若干规则，总体上可以用下表解释

{% asset_img Snipaste_2021-08-06_16-44-26.png %}

* 只有超级用户能够使用 setuid 设置实际用户ID和保存的设置用户ID；普通用户只能设置 有效用户ID
* 调用 exec 函数时，能否更改用户ID取决于文件的**设置用户ID位**的开闭
  * 设置用户ID位打开，则进程可以设置有效用户ID，保存的设置用户ID是有效用户ID 的拷贝值
  * 设置用户ID位关闭，则进程不能够设置有效用户ID



**seteuid 和 setegid 函数**

这俩函数只更改**有效用户ID和有效组ID**

```C
#include <unistd.h>

int seteuid(uid_t uid);

int setegid(gid_t gid);
//Both return: 0 if OK, −1 on error
```

各个设置ID函数的关系如下图

{% asset_img Snipaste_2021-08-06_16-51-12.png %}



## 8.12 解释器文件 interpreter file

所有现今的 UNIX 系统都支持解释器文件。这是一种文本文件，其**起始行**的形式是：

```shell
 #! pathname [optional-argument]
```

 

是否一定需要解释器文件？不一定。但是它能提高处理问题的速度。由于下述理由，解释器文件是有用的

* 有些程序是用某种语言写的脚本，解释器文件可以将这一事实隐藏起来
* 能够提高效率，减少 `fork` 、`exec` 和 `wait` 的开销
* 解释器脚本使我们可以使用除 /bin/sh 以外的其他 shell 来编写 shell 脚本



## 8.13 system 函数

```C
#include <stdlib.h>

int system(const char *cmdstring);
//Returns: (see below)
```

因为 `system` 函数调用了 `fork` 、 `exec` 和 `waitpid` ，因此有3种返回值

* `fork` 失败或者 `waitpid` 返回除 `EINTR` 之外的错误，则 `system` 返回 -1，并且设置 `errno` 以指示错误类型
*  如果 `exec` 失败，则其返回值如同 shell 执行了`exit(127)` 一样
* 否则，所有三个函数都成功执行，返回值是 shell 的终止状态

[system 函数的一种实现](https://github.com/imzhangjinming/APUE/blob/master/8/mysystem.c)



使用 `system` 而不是直接使用 `fork` 和 `exec` 的优点是： `system` 进行了所需的各种**出错 处理**及各种**信号处理**。





## 8.14 进程会计 process accounting

大多数UNIX系统提供了这项服务。启用该选项后，每当进程结束时内核就写一个**会计记录**。

会计记录结构定义在头文件 `<sys/acct.h>` 中，样式基本如下

```C
typedef u_short comp_t; /* 3-bit base 8 exponent; 13-bit fraction */
struct acct
{
	char	ac_flag;	/* flag (see Figure 8.26) */
	char	ac_stat;	/* termination status (signal & core flag only) */
    					/* (Solaris only) */
	uid_t ac_uid;		/* real user ID */
	gid_t ac_gid;		/* real group ID */
	dev_t ac_tty;		/* controlling terminal */
	time_t ac_btime;	/* starting calendar time */
	comp_t ac_utime;	/* user CPU time */
	comp_t ac_stime;	/* system CPU time */
	comp_t ac_etime;	/* elapsed time */
	comp_t ac_mem;		/* average memory usage */
	comp_t ac_io;		/* bytes transferred (by read and write) */
						/* "blocks" on BSD systems */
	comp_t ac_rw;		/* blocks read or written */
						/* (not present on BSD systems) */
	char	ac_comm[8]; /* command name: [8] for Solaris, */
						/* [10] for Mac OS X, [16] for FreeBSD, and */
						/* [17] for Linux */
};
```



## 8.15 用户标识

任一进程都可以得到其实际用户ID和有效用户ID及组ID。但是有时我们想要得到登录名。我们可以调用 `getpwuid(getuid())` ，但是如果一个用户有多个登录名，这些登录名又对应着同一个用户ID，又将如何呢？

我们可以使用 `getlogin` 获取登录名，然后使用 `getpwnam` 在口令文件中查找对应的用户，从而确定其登录 shell 等

```C
#include <unistd.h>

char *getlogin(void);
//Returns: pointer to string giving login name if OK, NULL on error
```



## 8.16 进程调度

进程可以通过调整其 nice 值选择以 **更低** 优先级运行（降低优先级也就降低了对CPU的占有，因此对 CPU 是 nice 的）。只有**特权进程**允许提高优先级。

SUS中 nice 值的范围在 0~(2*NZERO)-1 之间，nice 值越小，优先级越高（对CPU而言越不 nice）。`NZERO` 是系统定义的值，在Linux中可以使用 `sysconf(_SC_NZERO)` 来访问 `NZERO` 的值



进程只能通过 nice 函数调整自己的 nice 值，不能对其他任何进程的 nice 值产生影响

```C
#include <unistd.h>

int nice(int incr);
//Returns: new nice value − NZERO if OK, −1 on error
```



getpriority 函数可以像 nice 函数那样用于获取进程的 nice 值，但是它还可以获取一组相关进程 的 nice 值

```C
#include <sys/resource.h>

int getpriority(int which, id_t who);
//Returns: nice value between −NZERO and NZERO−1 if OK, −1 on error
```

setpriority 函数可用于为**进程、进程组**和属于**特定用户ID的所有进程**设置优先级

```C
#include <sys/resource.h>

int setpriority(int which, id_t who, int value);
//Returns: 0 if OK, −1 on error
```



## 8.17 进程时间

任一进程都可以调用 times 函数获得自己以及已终止子进程的运行时间

```C
#include <sys/times.h>

clock_t times(struct tms *buf );
//Returns: elapsed wall clock time in clock ticks if OK, −1 on error
```

times 函数填写 buf 指向的 tms 结构体变量，该结构体定义如下

```C
struct tms {
	clock_t tms_utime; 		/* user CPU time */
	clock_t tms_stime; 		/* system CPU time */
	clock_t tms_cutime; 	/* user CPU time, terminated children */
	clock_t tms_cstime; 	/* system CPU time, terminated children */
};
```



# 课后习题

## 3 重写 [这个程序] ，把 wait 换成 waitid。不调用 pr_exit，而从 siginfo 结构中确定等价的信息

[我的解答 新的打印终止状态的函数](https://github.com/imzhangjinming/APUE/blob/master/8/exercise/8.3.c)

[测试程序](https://github.com/imzhangjinming/APUE/blob/master/8/exercise/8.3_main.c)

