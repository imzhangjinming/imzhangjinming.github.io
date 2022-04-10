---
title: UNIX环境高级编程 第三章 -- 学习笔记
date: 2021-08-01 02:57:52
categories:
- notes
tags:
---

{% asset_img penguin-149971_1280.png %}

APUE 第三章 文件I/O



<!-- more -->

# 第三章 文件 I/O



 ## 3.2 文件描述符

内核通常需要同时打开多个文件，那么它如何区分各个不同的文件呢？使用它们的文件名吗？ 不是，内核使用**文件描述符**（file descriptor）来标识文件。文件描述符通常是一个非负整数。

按照惯例，shell 把文件描述符 **0** 与**标准输入**关联，**1**与**标准输出**关联，**2**与**标准错误**关联

在头文件 `<unistd.h>` 中，它们分别被定义为 `STDIN_FILENO` 、`STDOUT_FILENO` 、`STDERR_FILENO`



## 3.3 open 和 openat 函数

```c
#include<fcntl.h>

int open(const char *path, int oflag, .../* mode_t mode */);

int openat(int fd, const char *path, int oflag, .../* mode_t mode */);
//这两个函数，成功则返回文件描述符，失败则返回 -1
```



`oflag` 参数选项：

* **必须选且只能选一个的**

| 符号     | 意义               |
| -------- | ------------------ |
| O_RDONLY | 只读               |
| O_WRONLY | 只写               |
| O_RDWR   | 读写               |
| O_EXEC   | 只执行             |
| O_SEARCH | 只搜索（不太明白） |

* **可选的**

| 符号      | 意义                       |
| --------- | -------------------------- |
| O_APPEND  | 每次写的时候追加到文件末尾 |
| O_CLOEXEC |                            |
| O_CREAT   | 若文件不存在则创建它       |

{% asset_img Snipaste_2021-07-31_10-52-52.png %}

{% asset_img Snipaste_2021-07-31_10-53-32.png %}



`fd` **参数**把 `open` 和 `openat` 函数区分开来。有下面三种可能：

* `path` 指定了一个绝对路径，此时 `fd` 参数被忽略，两个函数完全相同

* `path` 指定了一个相对路径， `fd` 参数就是参照文件夹的文件描述符。

  比如我现在正在 `/home/ming/` 文件夹内，则相对路径 `../` 的参照文件夹就是我所在的文件夹， `fd` 就是我所在的文件夹的文件描述符

* `path` 指定了一个相对路径， `fd` 参数是 `AT_FDCWD` 。这种情况下，路径名在当前目录中获取。



**文件名和路径名截断**

若 `_POSIX_NO_TRUNC` 有效（使用之前介绍的检查实现限制的方法检查是否有效），那么当路径名超过 `PATH_MAX` 或文件名超过 `NAME_MAX` 时，系统会报错，并将 `errno` 设置为 `ENAMETOOLONG`



## 3.4 creat 函数

```C
#include<fcntl.h>

int creat(const char *path,mode_t mode);
//如果成功，返回文件描述符；失败则返回 -1
//creat 函数等效于：
open(path, O_WRONLY|O_CREAT|O_TRUNC, mode);
```



## 3.5 close 函数

```C
 #include<unistd.h>

int close(int fd);
// 如果成功，返回 0；失败则返回 -1
```



## 3.6 lseek 函数

每个打开的文件都有一个**当前文件偏移量**（current file offset），用以度量从文件开始处计算的字节数。

```C
 #include<unistd.h>

off_t lseek(int fd, off_t offset, int whence);
// 如果成功，返回新的文件偏移量；如果失败，返回 -1
```



`whence` 参数有三个选项：

* `SEEK_SET` 将偏移量设置为从距文件**开始**处 `offset` 个**字节**
* `SEEK_CUR` 将偏移量设置为从距文件**现在的偏移量**处 `offset` 个**字节**
* `SEEK_END` 将偏移量设置为从距文件**结尾**处 `offset` 个**字节**， `offset` 可正可负



[测试标准输入能否被设置偏移量](https://github.com/imzhangjinming/APUE/blob/master/3/set_stdinput_offset.c)



文件偏移量**大于**文件的**当前长度**，这是允许的。在这种情况下，对该文件哎拿到下一次写将加长该文件，并在文件中形成**空洞**，位于文件中但没有写过的字节被读为0



[创建一个具有空洞的文件](https://github.com/imzhangjinming/APUE/blob/master/3/file_with_hole.c)

`od(1)` 命令可以查看有空洞文件的具体内容



## 3.7 read 函数

```C
#include<unistd.h>

ssize_t read(int fd, void *buf, size_t nbytes);
//如果成功且未到文件结尾，返回读到的字节数；如果读到文件结尾，返回 0； 如果失败，返回 -1
```



## 3.8 write 函数

```C
#include<unistd.h>

ssize_t write(int fd, const void *buf, size_t nbytes);
//成功则返回写入的字节数；失败则返回 -1
```



为啥 `read` 和 `write` 函数的第二个参数有一个 `const` 的区别呢？

`read` 函数从文件内读取数据写入 `buf`指向的对象，所以不能有 `const`

`write` 函数从 `buf` 读取数据写入 `fd` 标识的文件，这里只对 `buf` 指向的对象执行读操作，所以要加 `const` 限制，防止 `buf` 指向的对象被更改 



## 3.10 文件共享

内核使用三种**数据结构**表示打开的文件：

* 每个进程在进程表中都有一个记录项，记录项中包含一张**打开文件的描述符表**，描述符表中是打开文件的描述符，和每个描述符关联的数据是：
  * 文件描述符标志
  * 指向一个**文件表项**的指针

* 内核为所有打开的文件维护一张**文件表**，每个**文件表项**（就是表中的每一项）包含：
  * 文件状态标志
  * 当前文件**偏移量**
  * 指向该文件 v 节点表项的指针

* 每个打开的文件都有一个 v 节点（v-node）结构。v 节点包含了文件类型和对此文件进行各种操作的函数的指针。通常情况下，v 节点还包含 i 节点（i-node）。



{% asset_img Snipaste_2021-07-31_17-54-49.png %}



如果有两个进程同时打开了同一文件，那么进程表项、文件表项、v-node 和 i-node之间的关系是这样的：

{% asset_img Snipaste_2021-07-31_17-58-07.png %}

可以看到，每个进程会指向自己的文件表项，文件表项中保存着**当前文件偏移量**，**文件状态标志**，然后它们同时指向该文件的 v-node 表项，这样不同的进程就能对同一文件进行操作哩



## 3.11 原子操作

能够保证读写操作原子性的俩个函数 `pread` 和 `pwrite`

```C
#include<unistd.h>

 ssize_t pread(int fd, void *buf, size_t nbytes, off_t offset);
//成功则返回读到的字节数，若已到文件末尾，则返回0；出错则返回 -1

ssize_t pwrite(int fd, const void *buf, size_t nbytes, off_t offset);
//成功则返回写的字节数；出错则返回 -1
```



## 3.12 dup 和 dup2 函数

这两个函数都可以用来复制一个现有的**文件描述符**

```C
#include<unistd.h>

int dup(int fd);

int dup2(int fd, int fd2);
// 成功则返回新的文件描述符；出错则返回 -1
```

这俩函数返回的新的文件描述符与原来的文件描述符**指向同一个文件表项**，看下图

{% asset_img Snipaste_2021-07-31_18-22-30.png %}



## 3.13 sync 、fsync 和 fdatasync 函数

在向文件内写入内容时，内核和磁盘之间设有缓冲区域，缓冲区域的目的是提高写入效率。但是缓冲区域的大小相对较小，当有其他内容需要写入磁盘时，缓冲区的内容会被替换掉。为了防止将没来得及写入磁盘的缓冲区内容清理掉，UNIX 提供了本小节标题里的三个函数

```C
 #include<unistd.h>

int fsync(int fd);

int fdatasync(int fd);
// 上面两个，如果成功返回 0； 出错则返回 -1

void sync(void);
```



## 3.14 fcntl  函数

可以**改变**已经打开的**文件的属性**

```C
#include<fcntl.h>

int fcntl(int fd, int cmd, .../*int arg*/);
//成功则依赖于 cmd；出错则返回 -1
```

下面的函数对一个文件描述符开启一个或多个文件状态标志

```C
#include"apue.h"
#include<fcntl.h>

void set_fl(int fd, int flags){
    int val;
    
    if((val=fcntl(fd,F_GETFL,0))<0)
        err_sys("fcntl F_GETFL error");
    
    val |= flags;
    
    if(fcntl(fd,F_SETFL,val)<0)
        err_sys("fcntl F_SETFL error");
}
```





## 3.16 /dev/fd

**打开文件 /dev/fd/n 相当于复制文件描述符 n**



# 课后习题



## 3.2 编写一个与 dup2 功能相同的函数，要求不调用fcntl函数，并且要有正确的出错处理

[解答](https://github.com/imzhangjinming/APUE/blob/master/3/exercise/mydup2.c)



## 3.6 如果使用追加标志打开一个文件以便读写，能否仍然使用lseek在任一位置开始读？能否使用lseek更新文件中任一部分的数据？请编程验证。

[解答](https://github.com/imzhangjinming/APUE/blob/master/3/exercise/3.6.c)

代码结果表明，可以任意读，但不能任意写
