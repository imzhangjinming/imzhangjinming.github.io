---
title: UNIX环境高级编程 第十五章 进程间通信
date: 2021-08-15 16:45:53
categories:
- notes
tags:
---


# 第十五章 InterProcess Communication , IPC

## 15.2 管道

管道是UNIX系统IPC的最古老形式，它有以下两种局限性

* 历史上，它们是半双工的
* 只能在具有**公共**祖先的两个进程之间使用

<!--more-->

每当在一个管道中键入一个命令序列，让 shell 执行，shell 都会为 每一条命令**单独创建** 一个进程，然后用管道将前一个进程的标准**输出**与后一个进程的标准**输入**相连

管道通过调用 `pipe` 函数创建

```C
#include <unistd.h>

int pipe(int fd[2]);
//Returns: 0 if OK, −1 on error
```

参数 `fd` 返回两个文件描述符，fd[1] 的输出是 fd[0] 的输入，它们之间的关系就像下图里描述的这样

 {% asset_img Snipaste_2021-08-14_09-42-33.png %}

管道是用来进行进程间通信的。通常的用法是先调用 `pipe` ，接着调用 `fork` ，从而创建从父进程到子进程 的IPC通道。下图显示了这种情况

{% asset_img Snipaste_2021-08-14_09-46-52.png %}

这一步之后，可以自由选择两个方向的数据流。一是从父进程到子进程的管道，也就是数据从父进程流向子进程

{% asset_img Snipaste_2021-08-14_09-48-09.png %}

父进程关闭 读端（fd[0]），子进程关闭 写端（fd[1]），注意，这里的 读/写 端是**针对管道**来说的，向管道写数据的就是写端，从管道读数据的就是读端。

二是从子进程流向父进程的数据流。读写段开闭情况与第一种相反。



[创建 由父进程流向子进程的管道](https://github.com/imzhangjinming/APUE/blob/master/15/pipe_parent2child.c)

[使用管道将文件复制到分页程序](https://github.com/imzhangjinming/APUE/blob/master/15/pipe_cp_file2more.c)



## 15.3 popen 和 pclose 函数

这两个函数实现的操作是：创建一个管道，`fork` 一个子进程，关闭未使用的管道端，执行一个 shell 命令，然后等待命令终止

```C
#include <stdio.h>

FILE *popen(const char *cmdstring, const char *type);
//Returns: file pointer if OK, NULL on error

int pclose(FILE *fp);
//Returns: termination status of cmdstring, or −1 on error
```

`popen` 先执行 `fork` ，然后调用 `exec` 执行 `cmdstring` ，并且返回一个标准 I/O 文件指针。

如果 `type` 是  `"r"` ，文件指针指向 `cmdstring` 的标准输出

如果 `type` 是  `"w"` ，文件指针指向 `cmdstring` 的标准输入

{% asset_img Snipaste_2021-08-14_10-30-12.png %}

{% asset_img Snipaste_2021-08-14_10-30-29.png %}



[使用 popen 将文件复制到分页程序](https://github.com/imzhangjinming/APUE/blob/master/15/popen_cp_file2more.c)

[popen 和 pclose 的一种实现](https://github.com/imzhangjinming/APUE/blob/master/15/popen_pclose.c)



## 15.4 协同进程 coprocesses

`popen` 只提供连接到另一个进程的标准输入或标准输出的**一个**单向管道，而协同进程则有连接到另一个进程的**两个**单向管道，一个接到标准输出，一个接到标准输入

{% asset_img Snipaste_2021-08-14_13-44-32.png %}



[使用 add2 协同进程 的例子](https://github.com/imzhangjinming/APUE/blob/master/15/add2_coprocess.c)



## 15.5 FIFO

FIFO 也被叫做 **命名管道** 。未命名的管道只能在**两个进程**之间使用，而且这两个进程还要有一个共同创建的它们的祖先进程。

通过FIFO，**不相关** 的进程也能交换数据。

FIFO 是一种 **文件类型** 。 通过 `stat` 结构的 `st_mode` 成员可以知道文件是否是 FIFO 类型。可以用 `S_ISFIFO` 宏对此进行测试。

创建 FIFO 也就像创建文件：

```C
#include <sys/stat.h>

int mkfifo(const char *path, mode_t mode);

int mkfifoat(int fd, const char *path, mode_t mode);
//Both return: 0 if OK, −1 on error
```

`mode` 参数可选值与 3.3 节中 `open` 函数 的 `mode` 参数可选值一样

新 FIFO 的用户和组的所有权规则与 4.6 节讲的一样



当用 `open` 打开一个 FIFO 时，非阻塞标志（`O_NONBLOCK`）会产生以下影响：

* 不指定 `O_NONBLOCK`，只读 `open` 要阻塞到某个其他进程为**写**而打开这个FIFO为止；只写 `open` 要阻塞到某个其他进程为读而打开它为止
* 指定了 `O_NONBLOCK`，则只读 `open` 立即返回。

类似于管道，若 `write` 一个**尚无进程为读而打开的FIFO**，则产生信号 `SIGPIPE `。若某个FIFO的最后一个写进程关闭了该FIFO，则将为该FIFO的读进程产生一个文件结束标志

常量 `PIPE_BUF` 说明了可以被原子地写到 FIFO 的最大数据量



FIFO 有以下两种用途：

* shell 命令使用FIFO 将数据从一条管道传送到另一条时，无需创建中间临时文件

  {% asset_img Snipaste_2021-08-14_15-54-15.png %}

* 客户-服务器进程应用程序中，FIFO用作汇聚点，在客户进程和服务器进程二者之间传递数据

  

## 15.6 XSI IPC

消息队列、信号量和共享存储器

### 15.6.1 标识符和键

标识符：非负整数，用于在内核中引用 IPC

键：IPC 的外部名，用于使多个进程能够使用同一 IPC 对象

从 键 到 标识符 的映射是**由内核完成**的



### 15.6.2 权限结构

XSI IPC 为每一个IPC结构关联了一个 `ipc_perm` 结构

```C
struct ipc_perm {
	uid_t 		uid;	/* owner’s effective user ID */
	gid_t 		gid;	/* owner’s effective group ID */
	uid_t 		cuid; 	/* creator’s effective user ID */
	gid_t 		cgid; 	/* creator’s effective group ID */
	mode_t 		mode; 	/* access modes */
	...
};
```





## 15.7 消息队列

消息队列就是消息的链表，由消息队列标识符（队列ID）标识。

 每个队列都有一个 `msqid_ds` 结构

```C
struct msqid_ds {
	struct ipc_perm 	msg_perm;		/* see Section 15.6.2 */
	msgqnum_t 			msg_qnum;		/* # of messages on queue */
	msglen_t 			msg_qbytes;   	/* max # of bytes on queue */
	pid_t 				msg_lspid;		/* pid of last msgsnd() */
	pid_t 				msg_lrpid;		/* pid of last msgrcv() */
	time_t 				msg_stime;		/* last-msgsnd() time */
	time_t 				msg_rtime;		/* last-msgrcv() time */
	time_t 				msg_ctime;		/* last-change time */
	...
};
```



`msgget` 用于创建新队列或打开一个现有队列

```C
#include <sys/msg.h>

int msgget(key_t key, int flag);
//Returns: message queue ID if OK, −1 on error
```

创建新队列时，需要初始化 `msqid-ds` 结构的下列成员

* `ipc-perm` 结构按 15.6.2 节中所述进行初始化
* `msg_qnum`, `msg_lspid`, `msg_lrpid`, `msg_stime`,  `msg_rtime`  全部设置为 0
* `msg_ctime` 设置为当前时间
* `msg_qbytes` 设置为系统限制值



`msgctl` 函数能对队列进行多种操作

```C
#include <sys/msg.h>

int msgctl(int msqid, int cmd, struct msqid_ds *buf );
//Returns: 0 if OK, −1 on error
```

`cmd `参数指定要对队列执行的命令

* `IPC_STAT` 取队列的 `msqid-ds` 结构并放到 `buf` 里
* `IPC_SET`  将队列的 `msg_perm.uid`, `msg_perm.gid`, `msg_perm.mode`, `msg_qbytes` 字段设置为 `buf `指定的值
* `IPC_RMID` 从系统中删除该消息队列以及仍在该队列中的所有数据



`msgsnd` 将队列添加到队列尾端

```C
#include <sys/msg.h>

int msgsnd(int msqid, const void *ptr, size_t nbytes, int flag);
//Returns: 0 if OK, −1 on error
```

`ptr` 是一个指针，它可以指向一个结构，结构内有一个长整型字段标识消息类型，还有真正要发送的消息（长度为 `nbytes`）。该结构体可以是下面这样的

```C
struct mymesg {
	long mtype;			/* positive message type */
	char mtext[512]; 	/* message data, of length nbytes */
};
```

`flag` 参数可以是 `IPC_NOWAIT` ，非阻塞，若消息队列已满则立即返回



`msgrcv` 从队列中取出消息

```C
#include <sys/msg.h>

ssize_t msgrcv(int msqid, void *ptr, size_t nbytes, long type, int flag);
//Returns: size of data portion of message if OK, −1 on error
```

`ptr` 指向一个长整型数，其后跟随的是存储实际数据的缓冲区。`nbytes` 指定缓冲区长度。

`type` 值指定想要那种消息

* `type==0` 返回队列中的第一个消息
* `type > 0` 返回队列中类型为 `type` 的第一个消息
* `type < 0` 返回队列中消息类型小于等于 `type` 绝对值的消息，如果这个消息有多个，则返回类型值最小的消息



## 15.8 信号量 semaphore

信号量和 管道、FIFO以及消息队列不同。它是一个计数器，用于为多个进程提供对**共享数据对象** 的访问



为了获得共享资源，进程需要执行下列操作

* 测试控制该资源的信号量
* 若信号量为正，进程可以使用该资源。这种情况下信号量减1
* 若信号量为 0 ，进程被阻塞直到信号量大于 0，然后从第一步开始执行



内核为每个**信号量集合**维护着一个 `semid_ds` 结构

```C
struct semid_ds {
	struct ipc_perm 	sem_perm; 	/* see Section 15.6.2 */
	unsigned short 		sem_nsems; 	/* # of semaphores in set */
	time_t 				sem_otime; 	/* last-semop() time */
	time_t 				sem_ctime; 	/* last-change time */
	...
};
```



每个**信号量**由一个无名结构表示，它至少包含下列成员

```C
struct {
	unsigned short 	semval;		/* semaphore value, always >= 0 */
	pid_t 			sempid;		/* pid for last operation */
	unsigned short	semncnt; 	/* # processes awaiting semval>curval */
	unsigned short	semzcnt; 	/* # processes awaiting semval==0 */
	...
};
```



想使用 XSI 信号量时，首先需要通过调用函数 `semget` 来获得一个信号量ID

```C
#include <sys/sem.h>

int semget(key_t key, int nsems, int flag);
//Returns: semaphore ID if OK, −1 on error
```

创建一个新集合时需要对 `semid_ds` 结构的下列成员赋值

* 按15.6.2 节初始化 `ipc_perm` 结构
* `sem_otime` 设置为 0
* `sem_ctime` 设置为当前时间
* `sem_nsems` 设置为 `nsems`



`semctl` 执行对信号量的操作

```C
#include <sys/sem.h>

int semctl(int semid, int semnum, int cmd, ... /* union semun arg */ );
//Returns: (see following)
```



`semop` 对信号量集合执行一系列操作

```C
#include <sys/sem.h>

int semop(int semid, struct sembuf semoparray[], size_t nops);
//Returns: 0 if OK, −1 on error
```



## 15.9 共享存储 shared memory

允许两个或多个进程共享一个给定的存储区（匿名的存储段），最快的一种IPC

内核为每个共享存储段维护着一个结构

```C
struct shmid_ds {
	struct ipc_perm 	shm_perm;	/* see Section 15.6.2 */
	size_t 				shm_segsz;	/* size of segment in bytes */
	pid_t 				shm_lpid;	/* pid of last shmop() */
	pid_t 				shm_cpid;	/* pid of creator */
	shmatt_t 			shm_nattch;	/* number of current attaches */
	time_t 				shm_atime;	/* last-attach time */
	time_t 				shm_dtime;	/* last-detach time */
	time_t 				shm_ctime;	/* last-change time */
	...
};
```



使用共享存储调用的第一个函数通常是 `shmget` ，它获得一个共享存储标识符

```C
#include <sys/shm.h>

int shmget(key_t key, size_t size, int flag);
//Returns: shared memory ID if OK, −1 on error
```

当创建一个新段时，初始化 `shmid_ds` 结构的下列成员

* `ipc_perm` 结构按 15.6.2 节初始化
* `shm_lpid`, `shm_nattch`, `shm_atime`, `shm_dtime` 全部初始化为 0
* `shm_ctime` 设置为当前时间
* `shm_segsz` 设置为请求的 `size`

参数 `size` 是请求的存储段长度，以字节为单位。



`shmctl` 函数对共享存储段进行多种操作

```C
#include <sys/shm.h>

int shmctl(int shmid, int cmd, struct shmid_ds *buf );
//Returns: 0 if OK, −1 on error
```

`cmd` 是下列五个值之一

* `IPC_STAT` 取此段的 `shmid_ds` 结构并存入 `buf`
* `IPC_SET` 将此段的 `shmid_ds` 结构设置为 `buf` 指定的内容
* `IPC_RMID` 从系统中删除此共享存储段，因为有 计数，所以直到最后一个使用该共享存储段的进程终止或与该段分离，此存储段才会真正被删除



以下操作不是 SUS 要求的，都只能由**超级用户**执行

* `SHM_LOCK` 对共享存储段加锁
* `SHM_UNLOCK` 对共享存储段解锁



创建了存储段后，进程可以使用 `shmat` 将其连接到它的地址空间中

```C
#include <sys/shm.h>

void *shmat(int shmid, const void *addr, int flag);
//Returns: pointer to shared memory segment if OK, −1 on error
```

`addr` 指定要连接到的地址，一般把它设置为 0 ，以便由系统自动选择



使用 `shmdt` 分离共享存储段

```C
#include <sys/shm.h>

int shmdt(const void *addr);
//Returns: 0 if OK, −1 on error
```



## 15.10 POSIX 信号量

解决了 XSI 信号量接口的几个缺陷：

* 更高性能
* 接口使用简单
* 删除操作更加完美



使用 `sem_open` 创建一个新的命名信号量或者使用一个现有的信号量

```C
#include <semaphore.h>

sem_t *sem_open(const char *name, int oflag, ... /* mode_t mode, unsigned int value */ );
//Returns: Pointer to semaphore if OK, SEM_FAILED on error
```

`oflag` 指定为 `O_CREAT` 时，`mode` 需要指定权限（和创建普通文件的权限设置方法相同），`value` 需要指定信号量的初始值（0~`SEM_VALLUE_MAX`）



命名信号量时最好遵循的一些规则

* 名字的第一个字符应该为 / 
* 名字不应该包含其它的 /
* 名字不应该长于 `_POSIX_NAME_MAX`



`sem_close` 释放与信号量相关的资源

```C
#include <semaphore.h>

int sem_close(sem_t *sem);
//Returns: 0 if OK, −1 on error
```



`sem_unlink` 销毁一个命名信号量

```C
#include <semaphore.h>

int sem_unlink(const char *name);
//Returns: 0 if OK, −1 on error
```



`sem_wait` 和 `sem_trywait` 函数对信号量 减 1

```C
#include <semaphore.h>

int sem_trywait(sem_t *sem);

int sem_wait(sem_t *sem);
//Both return: 0 if OK, −1 on error
```

如果信号量已经是 0 ，则 `sem_wait` 会阻塞直到成功减 1，`sem_trywait` 不会阻塞，它会立即返回 -1 ，并将 errno 设置为 `EAGAIN`

`sem_timedwait` 阻塞指定的时间

```C
#include <semaphore.h>
#include <time.h>

int sem_timedwait(sem_t *restrict sem,const struct timespec *restrict tsptr);
//Returns: 0 if OK, −1 on error
```



`sem_post` 使信号量值 加 1

```C
#include <semaphore.h>

int sem_post(sem_t *sem);
//Returns: 0 if OK, −1 on error
```



如果仅仅在单个进程中使用POSIX 信号量，那么使用未命名信号量更为容易。这仅仅改变创建和销毁信号量的方式

`sem_init` 函数创建一个未命名的信号量

```C
#include <semaphore.h>

int sem_init(sem_t *sem, int pshared, unsigned int value);
//Returns: 0 if OK, −1 on error
```

`pshared` 表明是否在多个进程中使用信号量，非零代表是，0 代表不是

`value` 指定信号量的初始值



`sem_destroy` 函数销毁未命名信号量

```C
#include <semaphore.h>

int sem_destroy(sem_t *sem);
//Returns: 0 if OK, −1 on error
```

`sem_getvalue` 获得信号量的值

```C
#include <semaphore.h>

int sem_getvalue(sem_t *restrict sem, int *restrict valp);
//Returns: 0 if OK, −1 on error
```



[使用二元信号量实现互斥量](https://github.com/imzhangjinming/APUE/blob/master/15/semaphores_mutex.c)



# 课后习题

## 1. 在 [此程序](https://github.com/imzhangjinming/APUE/blob/master/15/pipe_cp_file2more.c) 中，在父进程代码的末尾删除 waitpid 前的 close ，结果将如何？

分页程序能够正常读取指定文件，但读取完成后无法退出。因为父进程的写管道没有关闭，导致分页程序阻塞等待标准输入。

## 2. 在 [此程序](https://github.com/imzhangjinming/APUE/blob/master/15/pipe_cp_file2more.c) 中，在父进程代码的末尾删除 waitpid ，结果将如何？

由于父进程不再等待子进程终止，分页程序提前退出了

## 3. 如果popen 函数的参数是一个不存在的命令，会造成什么结果？

[测试程序](https://github.com/imzhangjinming/APUE/blob/master/15/exercise/15.3.c)

可见，`popen` 返回一个 非空指针，紧接着 输出 `sh: 1: meaninglesscmd: not found` 

`popen` 函数本身并不出错，它使用 shell 执行一个并不存在的命令，出错的是 shell 

## 5. 在 [此程序](https://github.com/imzhangjinming/APUE/blob/master/15/add2_coprocess.c) 中，用标准I/O库代替 read 和 write

[解答](https://github.com/imzhangjinming/APUE/blob/master/15/exercise/add2_coprocess.c)

