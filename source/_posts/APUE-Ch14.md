---
title: UNIX环境高级编程 第十四章 高级 I/O
date: 2021-08-13 20:38:16
categories:
- notes
tags:
---
{% asset_img 6EE6948103AA1311F3599FE082690801.png %}

梅老六远走巴黎



<!-- more -->

# 第十四章 高级 I/O

## 14.1 引言

本章将涵盖很多概念和函数：非阻塞I/O、记录锁、I/O多路转接、异步I/O、`readv` 和 `writev` 函数以及存储映射I/O



## 14.2 非阻塞I/O

在 10.5 节中曾把系统调用分为两类：低速系统调用和其他。低速系统调用可能会使进程永远阻塞，包括：

* 如果某些类型文件的数据不存在，则**读操作**可能会使调用者永远阻塞
* 如果写入的数据不能被文件直接接受，则**写操作**可能会永远阻塞调用者
* 打开特定文件类型可能会永远阻塞调用者，因为要等待特定的条件发生（例如要打开一个终端设备，需要先等待与之连接的调制解调器应答）
* `pause` 和 `wait` 函数
* 某些 `ioctl` 操作
* 某些 **进程间通信** 函数（见第15章）



非阻塞I/O使我们可以发出 `open` 、`read` 和 `write` 这样的 I/O 操作，并使这些操作 **不会** 永远阻塞。如果这种操作不能完成，则调用**立即出错返回**。

对于一个给定的文件描述符，有两种为其指定非阻塞I/O的方法

* 如果调用 `open` 获得描述符，则可以指定 `O_NONBLOCK` 标志
* 对于已经打开的描述符，调用 `fcntl` 打开 `O_NONBLOCK` 标志



[非阻塞 write](https://github.com/imzhangjinming/APUE/blob/master/14/nonblock_write.c)



## 14.3 记录锁 record locking

记录锁的功能是：当第一个进程正在读或者修改文件的某个部分时，使用记录锁可以阻止其他进程修改**同一文件区**

“记录”一词用在这里不太贴切，更合适的术语是 ***字节范围锁*** ，因为它锁定一定字节范围内的数据



**fcntl 记录锁**

```C
#include <fcntl.h>

int fcntl(int fd, int cmd, ... /* struct flock *flockptr */ );
//Returns: depends on cmd if OK (see following), −1 on error
```

对于记录锁，`cmd` 是 `F_GETLK`、`F_SETLK` 或 `F_SETLKW` ，第三个参数是一个指向 `flock` 结构的指针

```C
struct flock {
	short l_type;		/* F_RDLCK, F_WRLCK, or F_UNLCK */
	short l_whence; 	/* SEEK_SET, SEEK_CUR, or SEEK_END */
	off_t l_start; 		/* offset in bytes, relative to l_whence */
	off_t l_len;		/* length, in bytes; 0 means lock to EOF */
	pid_t l_pid;		/* returned with F_GETLK */
};
```

* `F_RDLCK` 共享读锁 、`F_WRLCK` 独占性写锁、`F_UNLCK` 解锁
* `l_start` 相对于 `l_whence` 的字节偏移量
* `l_len` 区域的字节长度
* `l_pid` 持有能够阻塞当前进程的锁的进程ID

说明

* 如果 `l_len` 为 0 ，则表示锁的范围可以扩展到最大可能偏移量。这意味着不管向该文件中追加写了多少数据，它们都可以处于锁的范围之内，而且起始位置可以是文件中的任意一个位置



`fcntl` 函数的三种命令

*  `F_GETLK` 判断由 `flockptr` 所描述的锁是否会被另外一把锁排斥。如果已经存在的锁阻止我们创建 `flockptr` 所描述的锁，那么 `fcntl` 会用现有的锁的信息重写 `flockptr` 指向的对象。如果不存在这种情况，则 `fcntl` 将 `flockptr` 指向的对象的 `l_type` 字段设置为 `F_UNLCK`
* `F_SETLK` 设置由  `flockptr` 所描述的锁。如果设置出错，则 `fcntl` 出错返回，`errno` 被设置为 `EACESS` 或 `EAGAIN`
*  `F_SETLKW` 这个命令是  `F_SETLK` 的**阻塞**版本。如果想要设置的锁因为兼容性问题不能完成，那么调用进程会休眠，直到想要的锁已经可以设置或者休眠状态被信号中断

[对文件某一区域 加锁/解锁 的函数](https://github.com/imzhangjinming/APUE/blob/master/14/lock_unlock_file.c)

[测试指定区域是否有锁的函数](https://github.com/imzhangjinming/APUE/blob/master/14/test_lock.c)



**锁的隐含继承和释放**

关于锁的自动继承和释放有3条规则：

* 锁 与进程 与文件 都相关联。

  第一，当一个进程终止时，它所建立的锁全部释放

  第二，无论一个描述符何时关闭，该进程通过这一描述符引用 的文件上的所有锁都会释放（这些锁都是该进程设置的）

* `fork` 产生的子进程不继承父进程设置的锁。这个约束是有道理的，锁的作用是防止多个进程同时写同一个文件。如果子进程通过 `fork` 继承父进程的锁，则父进程和子进程就能同时写同一个文件
* 在执行 `exec` 后，新程序可以继承原执行程序的锁。



## 14.4 I/O 多路转接 I/O multiplexing

### 14.4.1 select 和 pselect 函数

传给 `select` 的参数告诉内核：

* 我们关心的描述符
* 对于每个描述符我们关心的条件（读？写？是否关注一给定描述符的异常条件？）
* 愿意等待多长时间（可以永远等待、等待一固定时间或者根本不等待）

从 `select` 返回时，内核告诉我们：

* 已经准备好的描述符的总数量
* 对于读、写或异常这3个条件中的每一个，哪些描述符已经准备好

使用这些返回信息，就可以调用相应的I/O函数，并且确知该函数不会阻塞



```C
#include <sys/select.h>

int select(int maxfdp1, fd_set *restrict readfds,fd_set *restrict writefds, fd_set *restrict exceptfds,struct timeval *restrict tvptr);
//Returns: count of ready descriptors, 0 on timeout, −1 on error
```

`tvptr` 指定愿意等待的时间：

* `tvptr == NULL` 永远等待
* `tvptr->tv_sec==0 && tvptr->tv_usec==0` 根本不等待
* `tvptr->tv_sec != 0 || tvptr->tv_usec != 0` 等待指定的时间 



中间三个参数  `readfds`  `writefds`  `exceptfds`  是指向描述符集的指针

描述符集 `fd_set` 可用下面的函数进行设置

```C
#include <sys/select.h>

int FD_ISSET(int fd, fd_set *fdset);
//Returns: nonzero if fd is in set, 0 otherwise

void FD_CLR(int fd, fd_set *fdset);

void FD_SET(int fd, fd_set *fdset);

void FD_ZERO(fd_set *fdset);
```

`FD_ZERO` 将指定的描述符集清零

`FD_SET` 将指定的描述符加入到指定的描述符集中

`FD_CLR` 将指定的描述符从指定的描述符集中删除

`FD_ISSET` 测试指定描述符集中的指定描述符是否打开

在声明了一个描述符集变量之后，必须先将其清零，然后再对它进行相应操作



`select` 函数的第一个参数 `maxfdp1` 的意思是“最大文件描述符编号值加1” ，在参数指定的三个描述符集中找出最大描述符编号值，然后加1 ，就是第一个参数的值。这样，内核就不用在所有描述符范围内搜索，只需要在小于第一个参数的范围内搜索。



```C
#include <sys/select.h>

int pselect(int maxfdp1, fd_set *restrict readfds,fd_set *restrict writefds, fd_set *restrict exceptfds,const struct timespec *restrict tsptr,const sigset_t *restrict sigmask);
//Returns: count of ready descriptors, 0 on timeout, −1 on error
```

除了下列几点外，`pselect` 与 `select` 相同

* `select` 超时值用 `timeval` 指定，`pselect` 使用 `timespec` 结构
* `pselect` 的 `timespec` 参数被声明为 `const` ，表示它们不会在 `pselect` 函数内被修改
* `pselect` 可以使用可选信号屏蔽字



### 14.4.2 poll 函数

```C
#include <poll.h>

int poll(struct pollfd fdarray[], nfds_t nfds, int timeout);
//Returns: count of ready descriptors, 0 on timeout, −1 on error
```

与 `select` 不同的是，`poll` 通过 `pollfd` 类型的数组指定描述符和关心的条件。`fdarray` 数组中的元素个数由 `nfds` 指定，`timeout` 指定愿意等待的时间

```C
struct pollfd {
	int fd;			/* file descriptor to check, or <0 to ignore */
	short events; 	/* events of interest on fd */
	short revents; 	/* events that occurred on fd */
};
```

`events` 成员 设置为下图所示值的一个或几个

{% asset_img Snipaste_2021-08-13_12-16-42.png %}



## 14.5 异步 I/O

### 14.5.3 POSIX 异步 I/O

POSIX规定的异步 I/O 接口使用 AIO control block （AIO控制块）来描述 I/O 操作。 aiocb 结构定义了 AIO 控制块

```C
struct aiocb {
	int 			aio_fildes;		/* file descriptor */
	off_t 			aio_offset;		/* file offset for I/O */
	volatile void 	*aio_buf;		/* buffer for I/O */
	size_t			aio_nbytes;		/* number of bytes to transfer */
	int				aio_reqprio;	/* priority */
	struct sigevent aio_sigevent;	/* signal information */
	int				aio_lio_opcode;	/* operation for list I/O */
};
```

`aio_fildes` 文件描述符指定要进行读或写操作的文件；读写操作会从 `aio_offset` 指定的偏移量开始。

对于读操作，数据会被复制到 `aio_buf` 指定的缓冲区；对于写操作，数据会从 `aio_buf` 指定的缓冲区复制出来。

`aio_nbytes` 指定要读写的字节数。



注意，异步 I/O 并**不影响**由操作系统维护的文件偏移量

`aio_sigevent` 决定在 I/O 事件完成之后如何通知应用程序，通知方式由 `sigevent` 结构描述

```C
struct sigevent {
	int sigev_notify;								/* notify type */
	int sigev_signo;								/* signal number */
	union sigval sigev_value;						/* notify argument */
	void (*sigev_notify_function)(union sigval); 	/* notify function */
	pthread_attr_t *sigev_notify_attributes;		/* notify attrs */
};
```

`sigev_notify` 指定通知类型，取下面三个值之一

* `SIGEV_NONE` 		异步 I/O 请求完成后，不通知进程
* `SIGEV_SIGNAL`      异步 I/O 请求完成后，产生由 `sigev_signo` 指定的信号
* `SIGEV_THREAD`    异步 I/O 请求完成后，由 `sigev_notify_function` 指定的函数被调用，`sigev_value` 被作为它的唯一参数



在进行异步 I/O 之前需要先初始化 AIO 控制块。然后使用 `aio_read` 或 `aio_write` 函数进行异步读写操作

```C
#include <aio.h>

int aio_read(struct aiocb *aiocb);

int aio_write(struct aiocb *aiocb);
//Both return: 0 if OK, −1 on error
```



要想强制所有等待中的异步操作不等待而直接写入存储中，可以使用 `aio_fsync` 函数

```C
#include <aio.h>

int aio_fsync(int op, struct aiocb *aiocb);
//Returns: 0 if OK, −1 on error
```



`aio_error` 函数可以获得一个异步读、写或者同步操作的完成状态

```C
#include <aio.h>

int aio_error(const struct aiocb *aiocb);
//Returns: (see following)
```

返回值：

* 0 操作成功完成。需要调用 `aio_return` 函数获取操作返回值
* -1 对 `aio_error` 的调用失败
* `EINPROGRESS` 异步操作仍在等待中
* 其他情况 

如果异步操作成功，可以调用 `aio_return` 函数获取异步操作的返回值

```C
#include <aio.h>

ssize_t aio_return(const struct aiocb *aiocb);
//Returns: (see following)
```

注意，一定要在异步操作完成之后再调用 `aio_return` 函数，而且对每个异步操作只调用**一次**



执行 I/O 操作时，如果还有其他事务要处理而不想被  I/O 操作阻塞，就可以使用异步 I/O。但是如果所有其他事务都已经完成了，还有异步 I/O 操作没有完成，可以调用 `aio_suspend` 函数阻塞进程，直到异步操作完成

```C
#include <aio.h>

int aio_suspend(const struct aiocb *const list[], int nent,const struct timespec *timeout);
//Returns: 0 if OK, −1 on error
```



如果不想完成等待中的异步 I/O 操作时可以使用 `aio_cancel` 函数取消它们

```C
#include <aio.h>

int aio_cancel(int fd, struct aiocb *aiocb);
//Returns: (see following)
```

返回值：

* `AIO_ALLDONE`  操作在取消之前已经完成了
* `AIO_CANCELED` 所有操作已经取消
* `AIO_NOTCANCELED` 至少有一个要求取消的操作没有取消
* `-1` 对` aio_cancel` 调用失败



## 14.6 readv 和 writev 函数

这两个函数用来在一次函数调用中读、写多个**非连续**缓冲区。

```C
#include <sys/uio.h>

ssize_t readv(int fd, const struct iovec *iov, int iovcnt);

ssize_t writev(int fd, const struct iovec *iov, int iovcnt);
//Both return: number of bytes read or written, −1 on error
```

第二个参数 `iov` 是指向 `iovec` **结构数组**的一个指针

`iovec` 结构：

```C
struct iovec {
	void 	*iov_base;	/* starting address of buffer */
	size_t 	iov_len;	/* size of buffer */
};
```

`iovcnt` 指定 数组中元素的个数，其最大值受限于 `IOV_MAX`



下图显示了 `iov` 指向的数组结构与各个缓冲区的关系

{% asset_img Snipaste_2021-08-13_14-22-43.png %}

`writev` 函数会依次读取各个缓冲区的数据并输出，`readv` 函数会将读到的数据以此写入各个缓冲区



## 14.7 readn 和 writen 函数

管道、FIFO以及某些设备（特别是**终端**和**网络**）有下列两种性质

* 一次读操作所返回的数据可能少于所要求的数据，即使没有到达文件末尾也有可能发生。这不是一种错误，应该继续读
* 一次写操作的返回值可能少于指定输出的字节数。这也不是错误，应当继续写剩下的数据。



在读写管道、网络设备和终端时需要考虑这些特性。`readn` 和 `writen` 函数就是应用在这种场景中的，这两个函数可以多次调用 `read` 和 `write` 直到读写了指定数量的字节

```C
#include "apue.h"

ssize_t readn(int fd, void *buf, size_t nbytes);

ssize_t writen(int fd, void *buf, size_t nbytes);
//Both return: number of bytes read or written, −1 on error
```



[readn 和 writen 函数的一种实现](https://github.com/imzhangjinming/APUE/blob/master/14/readn_writen.c)



## 14.8 存储映射 I/O memory mapped I/O

存储映射 I/O 能将一个磁盘文件**映射**到存储空间的一个**缓冲区**上，于是，当从缓冲区中读取数据时，就相当于读文件中的相应字节；将数据存入缓冲区时，相应的字节就自动写入文件。

这样就可以在**不使用** `read` 和 `write` 的情况下执行 I/O

```C
#include <sys/mman.h>

void *mmap(void *addr, size_t len, int prot, int flag, int fd, off_t off );
//Returns: starting address of mapped region if OK, MAP_FAILED on error
```

`addr` 指定缓冲区的起始地址。将其设置为0可以让系统自动选择缓冲区起始地址，该函数的返回值就是缓冲区的起始地址

`fd` 参数指定要被映射的文件的描述符

`len` 是要映射的字节数

`off` 是起始偏移量，从它开始 的 `len` 字节将被映射

`prot` 参数指定映射存储区的保护要求，如下图所示

{% asset_img Snipaste_2021-08-13_14-53-12.png %}

下图显示了文件和映射缓冲区的关系

{% asset_img Snipaste_2021-08-13_14-55-03.png %}

`flag` 参数影响映射存储区的属性

* `MAP_FIXED` 不利于可移植性，不建议使用
* `MAP_SHARED`  对映射缓冲区的操作就是对被映射文件的操作
* `MAP_PRIVATE` 对缓冲区的操作会导致创建被映射文件的**一个副本**



调用 `mprotect` 可以更改一个现有映射的**权限**

```C
#include <sys/mman.h>

int mprotect(void *addr, size_t len, int prot);
//Returns: 0 if OK, −1 on error
```



如果共享映射中的页已修改，那么可以调用 `msync` 将该页冲洗到被映射的文件中

```C
#include <sys/mman.h>

int msync(void *addr, size_t len, int flags);
//Returns: 0 if OK, −1 on error
```

`flags` 必须选择 `MS_ASYNC` 、 `MS_SYNC` 两者中的一个



使用 `munmap` 函数解除映射，仅仅关闭文件描述符并不解除映射

```C
#include <sys/mman.h>

int munmap(void *addr, size_t len);
//Returns: 0 if OK, −1 on error
```



