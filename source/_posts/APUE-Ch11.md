---
title: UNIX环境高级编程 第十一章 线程
date: 2021-08-10 15:21:54
categories:
- notes
tags:
---
{% asset_img needle-4854847_1920.jpg %}

APUE 第十一章 线程



<!-- more -->

# 第十一章 线程 thread

## 11.2 线程概念

一个进程的所有信息对该进程的所有线程都是共享的，包括可执行程序的代码、程序的全局内存和堆内存、栈以及文件描述符

 APUE讨论的线程接口来自 POSIX.1-2001。线程接口也称为 pthread 或 POSIX线程

POSIX线程的功能测试宏是 `_POSIX_THREADS `，可以用这个宏测试是否支持线程



## 11.3 线程标识

每个线程也有自己的线程ID。但是和 进程ID 的全局唯一性不同，线程ID只在自己的进程环境内有意义。

线程ID的数据类型是 `pthread_t` ，它有可能是整数类型，也有可能是结构体，所以不能直接使用 `=` 判断两个线程ID是否相等，得使用下面这个函数

```C
#include <pthread.h>

int pthread_equal(pthread_t tid1, pthread_t tid2);
//Returns: nonzero if equal, 0 otherwise
```



也没有可移植的打印线程ID的方法（因为线程ID的数据类型随实现的不同而不同）

线程可以调用 `pthread_self` 函数获得自身的线程ID

```C
#include <pthread.h>

pthread_t pthread_self(void);
//Returns: the thread ID of the calling thread
```



[打印进程ID和线程ID](https://github.com/imzhangjinming/APUE/blob/master/11/pr_tid.c)



## 11.4 创建线程

```C
#include <pthread.h>

int pthread_create(pthread_t *restrict tidp,const pthread_attr_t *restrict attr,void *(*start_rtn)(void *), void *restrict arg);
//Returns: 0 if OK, error number on failure
```

成功返回时，新线程的线程ID被储存在 `tidp` 指向的内存中 。`attr` 参数用来定制不同的线程属性，将其置为 `NULL` 可以创建具有默认属性的线程

新创建的线程从 `start_rtn` 函数的地址开始运行，`arg` 是此函数的参数

注意，无法确定新创建的线程和调用线程哪个先执行。新线程可以访问进程的地址空间，继承调用线程的浮点环境和信号屏蔽字，但是该线程的挂起信号集会被清除



## 11.5 终止线程

如果进程中的任意线程调用了 `exit` 、`_Exit` 或 `_exit` ，那么整个进程都会终止。如果一个线程接收到默认动作是终止进程的信号，那么整个进程也会终止。



线程也可以自己退出而不终止整个进程

* 从启动函数中返回，返回值是线程的**退出码**
* 被**同一进程**中的其他线程取消
* 调用 `pthread_exit` 



```C
#include <pthread.h>

void pthread_exit(void *rval_ptr);
```

`rval_ptr` 是一个无类型指针，进程中的其他线程可以使用 `pthread_join` 函数访问这个指针，其中包含了线程的返回码（从启动函数返回）或者返回状态

```C
#include <pthread.h>

int pthread_join(pthread_t thread, void **rval_ptr);
//Returns: 0 if OK, error number on failure
```



[获得线程的退出状态码](https://github.com/imzhangjinming/APUE/blob/master/11/get_thread_exit_status.c)



线程可以通过调用 `pthread_cancel` 函数来请求**取消**同一进程中的**其他线程**

```C
#include <pthread.h>

int pthread_cancel(pthread_t tid);
//Returns: 0 if OK, error number on failure
```

此函数使得 `tid` 指定的线程的行为和调用了参数为 `PTHREAD_CALCELED` 的 `pthread_exit` 函数的线程一样



线程可以指定它退出时需要调用的函数，类似 `atexit` 函数，这样的程序被称为 thread cleanup handler 

一个线程可以有多个清理程序，它们被装在栈中，自然地，它们被执行的顺序与注册的顺序相反（栈的特性决定）

```C
#include <pthread.h>

void pthread_cleanup_push(void (*rtn)(void *), void *arg);

void pthread_cleanup_pop(int execute);
```

当线程执行以下动作时，清理函数被调用

* 调用 `pthread_exit` 
* 响应取消请求
* 以非零 `execute` 参数调用 `pthread_cleanup_pop` 函数。以非零 `execute` 参数调用 `pthread_cleanup_pop` ，最后压入栈的清理函数被执行；如果 `execute=0` ，最后压入栈的函数被弹出而不执行



下图显示了进程和线程函数的相似之处

{% asset_img Snipaste_2021-08-09_16-19-53.png %}



`pthread_detach` 函数用来分离线程

```C
#include <pthread.h>

int pthread_detach(pthread_t tid);
//Returns: 0 if OK, error number on failure
```



## 11.6 线程同步

当多个线程都能读取或写某些相同的变量时，就会产生一致性问题，就需要对这些线程进行同步，确保不同线程在访问变量的存储内容时不会访问到无效的值

线程使用**锁**解决这个问题，同一时间只允许一个线程访问该变量。



### 11.6.1 互斥量 mutex 

互斥量从本质上来讲是一把锁，在访问共享资源**前**对互斥量进行设置，访问完成后释放互斥量。

互斥量使用 `pthread_mutex_t` 数据类型表示的。此数据类型要求在使用前进行初始化。

```C
#include <pthread.h>

int pthread_mutex_init(pthread_mutex_t *restrict mutex,const pthread_mutexattr_t *restrict attr);

int pthread_mutex_destroy(pthread_mutex_t *mutex);
//Both return: 0 if OK, error number on failure
```

`attr` 参数用来指定互斥量的属性。如果互斥量是**动态分配**的，则释放内存前要调用 `pthread_mutex_destroy`



```C
#include <pthread.h>

int pthread_mutex_lock(pthread_mutex_t *mutex);

int pthread_mutex_trylock(pthread_mutex_t *mutex);

int pthread_mutex_unlock(pthread_mutex_t *mutex);
//All return: 0 if OK, error number on failure
```

`pthread_mutex_lock` 对互斥量上锁。如果互斥量已经上锁，则线程被阻塞

`pthread_mutex_unlock` 对互斥量解锁

`pthread_mutex_trylock` 尝试对互斥量进行上锁。如果互斥量没有上锁，调用此函数将互斥量上锁；如果互斥量已经上锁，此调用会失败，返回 `EBUSY `，线程不会阻塞



[使用互斥锁](https://github.com/imzhangjinming/APUE/blob/master/11/use_mutex.c)



### 11.6.2 避免死锁

### 11.6.3 pthread_mutex_timedlock 函数

这个函数允许指定一个时间（绝对的时间），如果在指定的时间内互斥量可以上锁，则上锁；如果超出指定时间，函数不会再对互斥量上锁，而是返回错误码 `ETIMEDOUT`

```C
#include <pthread.h>
#include <time.h>

int pthread_mutex_timedlock(pthread_mutex_t *restrict mutex,const struct timespec *restrict tsptr);
//Returns: 0 if OK, error number on failure
```



### 11.6.4 读写锁 reader-writer lock

读写锁有三种状态

* 读模式下加锁状态
* 写模式下加锁状态
* 不加锁状态

一次只有**一个**线程可以占有**写模式**的读写锁，多个线程可以同时占有读模式的读写锁



读写锁是**写加锁**状态时会**阻塞一切**试图对这个锁进行加锁的线程

读写锁是**读加锁**状态时会**阻塞**一切试图随这个锁进行**写加锁**的线程，试图读加锁的线程可以获得访问权



读写锁在使用之前必须**初始化**，释放内存之前必须**销毁**

```C
#include <pthread.h>

int pthread_rwlock_init(pthread_rwlock_t *restrict rwlock,const pthread_rwlockattr_t *restrict attr);

int pthread_rwlock_destroy(pthread_rwlock_t *rwlock);
//Both return: 0 if OK, error number on failure
```



加锁与解锁函数：

```C
#include <pthread.h>

int pthread_rwlock_rdlock(pthread_rwlock_t *rwlock);	/* 读加锁 */

int pthread_rwlock_wrlock(pthread_rwlock_t *rwlock);	/* 写加锁 */

int pthread_rwlock_unlock(pthread_rwlock_t *rwlock);
//All return: 0 if OK, error number on failure
```



[使用读写锁实现任务管理](https://github.com/imzhangjinming/APUE/blob/master/11/use_reader_writer_lock.c)



### 11.6.5 带有超时的读写锁

```C
#include <pthread.h>
#include <time.h>

int pthread_rwlock_timedrdlock(pthread_rwlock_t *restrict rwlock,const struct timespec *restrict tsptr);

int pthread_rwlock_timedwrlock(pthread_rwlock_t *restrict rwlock,const struct timespec *restrict tsptr);
//Both return: 0 if OK, error number on failure
```

这俩函数的作用和 `pthread_mutex_timedlock` 函数的作用类似



### 11.6.6 条件变量

这是线程可用的另一种**同步机制** 。它本身是由互斥量保护的。

条件变量的数据类型是 `pthread_cond_t` ，它有两种初始化方法（必须初始化才能使用）

* 静态分配，可以将 `PTHREAD_COND_INITIALIZER` 赋值给条件变量
* 动态分配， 调用 `pthread_cond_init` 函数

```C
include <pthread.h>
    
int pthread_cond_init(pthread_cond_t *restrict cond,const pthread_condattr_t *restrict attr);

int pthread_cond_destroy(pthread_cond_t *cond);
//Both return: 0 if OK, error number on failure
```



我们使用 `pthread_cond_wait` 函数等待 条件变量变为真

```C
#include <pthread.h>

int pthread_cond_wait(pthread_cond_t *restrict cond,pthread_mutex_t *restrict mutex);

int pthread_cond_timedwait(pthread_cond_t *restrict cond,pthread_mutex_t *restrict mutex,const struct timespec *restrict tsptr);
//Both return: 0 if OK, error number on failure
```

传递给  `pthread_cond_wait` 函数的互斥量对条件进行保护。调用者把**锁住**的互斥量传递给函数，函数然后自动把调用线程放到等待条件的线程列表上，对互斥量解锁。这样就关闭了条件检查和线程进入休眠等待条件改变这两个操作之间的时间通道，线程从而不会错过条件的任何变化。

 `pthread_cond_wait` 返回时，互斥量被再次锁住



`pthread_cond_signal` 函数可以通知**至少一个**线程条件已经满足

`pthread_cond_broadcast` 函数能够唤醒等待该条件的**所有进程**

```C
#include <pthread.h>

int pthread_cond_signal(pthread_cond_t *cond);

int pthread_cond_broadcast(pthread_cond_t *cond);
//Both return: 0 if OK, error number on failure
```

[使用条件变量的例子](https://github.com/imzhangjinming/APUE/blob/master/11/use_cond_var.c)



### 11.6.7 自旋锁

自旋锁通常作为底层原语用于实现其他类型的锁



### 11.6.8 屏障 barrier

用户协调多个线程并行工作的同步机制。

屏障允许线程等待，知道所有的合作线程都到达某一点，然后从该点继续执行

```C
#include <pthread.h>

int pthread_barrier_init(pthread_barrier_t *restrict barrier,const pthread_barrierattr_t *restrict attr,unsigned int count);

int pthread_barrier_destroy(pthread_barrier_t *barrier);
//Both return: 0 if OK, error number on failure
```

`pthread_barrier_init `用于初始化屏障，`pthread_barrier_destroy` 用于反初始化屏障



```C
#include <pthread.h>

int pthread_barrier_wait(pthread_barrier_t *barrier);
//Returns: 0 or PTHREAD_BARRIER_SERIAL_THREAD if OK, error number on failure
```

`pthread_barrier_wait` 函数表明线程已经完成工作，等待其他线程赶上来

[使用多线程和屏障进行数组排序](https://github.com/imzhangjinming/APUE/blob/master/11/use_barrier.c)



# 课后习题

## 1. 修改[这个例子](https://github.com/imzhangjinming/APUE/blob/master/11/incorrect_use_pthread_exit.c)，正确地在两个线程之间传递结构

[我的解答](https://github.com/imzhangjinming/APUE/blob/master/11/exercise/correct_use_pthread_exit.c)

