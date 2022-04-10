---
title: UNIX环境高级编程 第十二章 控制线程
date: 2021-08-10 23:02:19
categories:
- notes
tags:
---

{% asset_img needle-4854847_1920.jpg %}

APUE 第十二章 线程控制



<!-- more -->

# 第十二章 线程控制

## 12.2 线程限制

SUS定义了与线程操作有关的一些限制，可以使用 `sysconf` 函数查询它们，下表总结了这些限制

{% asset_img Snipaste_2021-08-10_10-42-58.png %}



## 12.3 线程属性

用来细调和定制线程的东西，数据类型是 `pthread_attr_t`

管理这些属性的函数都遵循同样的模式：

* 每个对象与它自己类型的**属性对象**进行关联。一个属性对象可以代表多个属性。属性对象对应用程序来说是**不透明的**
* 有一个初始化函数，把属性设置为默认值
* 有一个销毁属性对象的函数
* 每个属性都有一个从属性对象中获取属性值的函数
* 每个属性都有一个设置属性值的函数



使用以下两个函数初始化和销毁属性对象

```C
#include <pthread.h>

int pthread_attr_init(pthread_attr_t *attr);

int pthread_attr_destroy(pthread_attr_t *attr);
//Both return: 0 if OK, error number on failure
```



**分离线程**

不需要取得退出码，希望线程一终止就立即回收其占有的资源的线程

可以使用 `pthread_detach` 函数分离一个线程

```C
#include <pthread.h>

int pthread_detach(pthread_t tid);
//Returns: 0 if OK, error number on failure
```



使用以下两个函数获得或修改一个属性对象的分离线程属性

```C
#include <pthread.h>

int pthread_attr_getdetachstate(const pthread_attr_t *restrict attr,int *detachstate);

int pthread_attr_setdetachstate(pthread_attr_t *attr, int detachstate);
//Both return: 0 if OK, error number on failure
```

分离状态要么是 `PTHREAD_CREATE_DETACHED` （分离状态） 要么是 `PTHREAD_CREATE_JOINABLE`（正常状态）



[以分离状态创建线程](https://github.com/imzhangjinming/APUE/blob/master/12/create_detached_thread.c)



**线程栈属性**

遵循POSIX标准的实现不一定支持线程栈属性，但是遵循SUS中XSI选项的系统来说，支持线程栈属性是必须的。

可以使用 `_POSIX_THREAD_ATTR_STACKADDR` 和 `_POSIX_THREAD_ATTR_STACKSIZE` 检查实现是否支持线程栈属性（在编译时或运行时检测）



使用以下两个函数对线程栈属性进行管理

```C
#include <pthread.h>

int pthread_attr_getstack(const pthread_attr_t *restrict attr,void **restrict stackaddr,size_t *restrict stacksize);

int pthread_attr_setstack(pthread_attr_t *attr,void *stackaddr, size_t stacksize);
//Both return: 0 if OK, error number on failure
```

`stackaddr` 指定栈的**最低**内存地址（注意不一定是栈的开始位置），`stacksize` 指定栈的大小



如果想改变线程栈的大小，又不想管理栈内存的分配问题，那么下面这两个函数就会很有用

```C
#include <pthread.h>

int pthread_attr_getstacksize(const pthread_attr_t *restrict attr,size_t *restrict stacksize);

int pthread_attr_setstacksize(pthread_attr_t *attr, size_t stacksize);
//Both return: 0 if OK, error number on failure
```

注意：设置的 `stacksize` 不能小于 `PTHREAD_STACK_MIN` 



## 12.4 同步属性

### 12.4.1 互斥量属性

数据类型 `pthread_mutexattr_t`

初始化和销毁

```C
#include <pthread.h>

int pthread_mutexattr_init(pthread_mutexattr_t *attr);

int pthread_mutexattr_destroy(pthread_mutexattr_t *attr);
//Both return: 0 if OK, error number on failure
```

 互斥量属性中有一个 **进程共享属性** ，它决定互斥量是否可以用来保护在**多个进程之间**共享的数据

* `PTHREAD_PROCESS_PRIVATE` 默认的，不可以保护进程共享的数据
* `PTHREAD_PROCESS_SHARED` 可以保护进程共享的数据



使用以下函数查询或更改 进程共享属性

```C
#include <pthread.h>
int pthread_mutexattr_getpshared(const pthread_mutexattr_t *restrict attr,int *restrict pshared);

int pthread_mutexattr_setpshared(pthread_mutexattr_t *attr,int pshared);
//Both return: 0 if OK, error number on failure
```



另一种互斥量属性： **互斥量类型**

它有以下四种值

| 值                         | 说明                                                         |
| -------------------------- | ------------------------------------------------------------ |
| `PTHREAD_MUTEX_NORMAL`     | A standard mutex type that doesn’t do any special error checking or deadlock detection. |
| `PTHREAD_MUTEX_ERRORCHECK` | A mutex type that provides error checking.                   |
| `PTHREAD_MUTEX_RECURSIVE`  | A mutex type that allows the same thread to lock it multiple times without first unlocking it. A recursive mutex maintains a lock count and isn’t released until it is unlocked the same number of times it is locked. Thus, if you lock a recursive mutex twice and then unlock it, the mutex remains locked until it is unlocked a second time. |
| `PTHREAD_MUTEX_DEFAULT`    | A mutex type providing default characteristics and behavior. Implementations are free to map it to one of the other mutex types. For example, Linux 3.2.0 maps this type to the normal mutex type, whereas FreeBSD 8.0 maps it to the error- checking type. |

该属性通过以下函数管理

```C
#include <pthread.h>

int pthread_mutexattr_gettype(const pthread_mutexattr_t *restrict attr, int *restrict type);

int pthread_mutexattr_settype(pthread_mutexattr_t *attr, int type);
//Both return: 0 if OK, error number on failure
```



### 12.4.2 读写锁属性

数据类型 `pthread_rwlockattr_t`

```C
#include <pthread.h>

int pthread_rwlockattr_init(pthread_rwlockattr_t *attr);

int pthread_rwlockattr_destroy(pthread_rwlockattr_t *attr);
//Both return: 0 if OK, error number on failure
```

读写锁单纯得多，它只支持 **进程共享属性**。这个属性的解释和互斥量的进程共享属性是相同的

```C
#include <pthread.h>

int pthread_rwlockattr_getpshared(const pthread_rwlockattr_t *restrict attr,int *restrict pshared);

int pthread_rwlockattr_setpshared(pthread_rwlockattr_t *attr,int pshared);
//Both return: 0 if OK, error number on failure
```



### 12.4.3 条件变量属性

数据类型 `pthread_condattr_t`

SUS定义条件变量支持 **进程共享** 和 **时钟** 两个属性

```C
#include <pthread.h>

int pthread_condattr_init(pthread_condattr_t *attr);

int pthread_condattr_destroy(pthread_condattr_t *attr);
//Both return: 0 if OK, error number on failure
```



关于  **进程共享属性** 

```C
include <pthread.h>
    
int pthread_condattr_getpshared(const pthread_condattr_t *restrict attr,int *restrict pshared);

int pthread_condattr_setpshared(pthread_condattr_t *attr,int pshared);
//Both return: 0 if OK, error number on failure
```



关于 **时钟属性**

这个属性控制计算 `pthread_cond_timedwait` 是否超时时用的是哪个时钟

```C
#include <pthread.h>

int pthread_condattr_getclock(const pthread_condattr_t *restrict attr,clockid_t *restrict clock_id);

int pthread_condattr_setclock(pthread_condattr_t *attr,clockid_t clock_id);
//Both return: 0 if OK, error number on failure
```



### 12.4.4 屏障属性

数据类型 `pthread_barrierattr_t`

```C
#include <pthread.h>

int pthread_barrierattr_init(pthread_barrierattr_t *attr);

int pthread_barrierattr_destroy(pthread_barrierattr_t *attr);
//Both return: 0 if OK, error number on failure
```

只支持 **进程共享属性**

```C
#include <pthread.h>

int pthread_barrierattr_getpshared(const pthread_barrierattr_t *restrict attr,int *restrict pshared);

int pthread_barrierattr_setpshared(pthread_barrierattr_t *attr,int pshared);
//Both return: 0 if OK, error number on failure
```



## 12.5 重入

如果一个函数在同一时间可以被**多个线程**安全地调用，就称该函数是线程安全的

支持线程安全函数的操作系统会定义 `_POSIX_THREAD_SAFE_FUNCTIONS` 符号

除了下表中列出的函数，SUS保证它定义的所有其它函数都是线程安全的

{% asset_img Snipaste_2021-08-10_12-20-25.png %}



一个函数 **线程安全** 版本和 **线程不安全** 版本的对比

[getenv 线程不安全版本](https://github.com/imzhangjinming/APUE/blob/master/12/mygetenv.c)

[getenv  线程安全版本](https://github.com/imzhangjinming/APUE/blob/master/12/mygetenv_r.c)

可以看到，为了线程安全要付出很多额外的努力



## 12.6 线程特有数据 thread-specific data

也称为线程私有数据 thread-private data ，是存储和查询某个特定线程相关数据的一种机制

在分配线程特定数据之前，需要创建一个 与该数据关联的 **键** 。这个键将用于获取对线程特定数据的访问

```C
#include <pthread.h>

int pthread_key_create(pthread_key_t *keyp, void (*destructor)(void *));
//Returns: 0 if OK, error number on failure
```

新创建的键保存在 `keyp` 中，**每个线程**都可以使用这个键，但是对于每个线程这个键都**映射到**不同的私有数据**地址**



线程通常使用 `malloc` 为特有数据分配内存。可以在创建 键 的时候指定一个析构函数，当线程调用 `pthread_exit` 或者线程执行返回，正常退出时，析构函数就会被调用。



对于所有线程，我们都可以调用 `pthread_key_delete` 来取消键与线程特有数据之间的关联关系

```C
#include <pthread.h>

int pthread_key_delete(pthread_key_t key);
//Returns: 0 if OK, error number on failure
```



```C
#include <pthread.h>

pthread_once_t initflag = PTHREAD_ONCE_INIT;

int pthread_once(pthread_once_t *initflag, void (*initfn)(void));
//Returns: 0 if OK, error number on failure
```

`initflag` 必须是一个非本地变量（静态变量或者全局变量），必须初始化为 `PTHREAD_ONCE_INIT`

通过 `pthread_once` ，系统可以保证 函数 `initfn` 只调用一次



```C
#include <pthread.h>

void *pthread_getspecific(pthread_key_t key);
/*
 * Returns: thread-specific data value or NULL if no value
 * has been associated with the key
 */

int pthread_setspecific(pthread_key_t key, const void *value);
//Returns: 0 if OK, error number on failure
```



## 12.7 取消选项 cancel options 

有两个线程属性是没有包含在 `pthread_attr_t` 结构中的：***可取消状态***  和 ***可取消类型***

这两个属性影响线程如何响应 `pthread_cancel` 函数



***可取消状态***  `PTHREAD_CANCEL_ENABLE` 或者 `PTHREAD_CANCEL_DISABLE`

```C
#include <pthread.h>

int pthread_setcancelstate(int state, int *oldstate);
//Returns: 0 if OK, error number on failure
```

线程调用  `pthread_cancel`  函数后并不会立即停止，而是在下一个 *取消点*  检查线程是否被取消，如果取消了，就执行取消响应。

在调用下面这些函数的时候一定会出现取消点

{% asset_img Snipaste_2021-08-10_16-11-40.png %}



## 12.8 线程和信号

可以想象，引入线程后，信号处理变得更加复杂了

首先需要明确的是，每个线程都有自己的**信号屏蔽字**，但是对信号的**处理方式**是进程中所有线程**共享**的



线程使用 `pthread_sigmask` 设置自己的信号屏蔽字

```C
#include <signal.h>

int pthread_sigmask(int how, const sigset_t *restrict set,sigset_t *restrict oset);
//Returns: 0 if OK, error number on failure
```

使用 `sigwait` 等待信号出现，注意为了避免出错，线程在调用 `sigwait` 之前，必须**阻塞**那些它正在等待的**信号**

```C
#include <signal.h>

int sigwait(const sigset_t *restrict set, int *restrict signop);
//Returns: 0 if OK, error number on failure
```

使用 `pthread_kill` 把信号发送给线程

```C
#include <signal.h>

int pthread_kill(pthread_t thread, int signo);
//Returns: 0 if OK, error number on failure
```



## 12.9 线程和 fork 

当线程调用 `fork` 时，就为子进程创建了整个进程地址空间的副本。

子进程通过继承整个地址空间的副本，还从父进程继承了每个互斥量、读写锁和条件变量的状态。

即使父进程有多个线程，子进程也只有一个线程，这个线程由父进程中调用 `fork` 的那个线程的副本构成。这样就产生了一个问题：子进程继承了父进程的互斥量、读写锁和条件变量的状态，但是子进程没有继承相应的线程，也就无法知道它占有了哪些锁、需要释放哪些锁

要解决这个问题，就要在 `fork` 的时候清除锁状态。可以调用 `pthread_atfork` 函数建立 `fork` 处理程序

```C
#include <pthread.h>

int pthread_atfork(void (*prepare)(void), void (*parent)(void),void (*child)(void));
//Returns: 0 if OK, error number on failure
```

`prepare` 由父进程在 `fork` 创建子进程 之前调用，它获得父进程定义的 **所有锁**。

`parent` 在 `fork` 创建子进程之后、返回之前在**父进程**上下文中调用，它对 `prepare` 程序获得的所有锁 **解锁**

`child` 在 `fork` 创建子进程之后、返回之前在 **子进程**上下文中调用，它也对 `prepare` 程序获得的所有锁进行**解锁**

[使用 pthread_atfork](https://github.com/imzhangjinming/APUE/blob/master/12/use_pthread_atfork.c)



## 12.10 线程和 I/O

首先需要记住的是进程中的所有线程**共享文件描述符**

多线程 I/O 时，`pread`  和 `pwrite` 对于解决并发线程对同一文件进行读写操作的问题



