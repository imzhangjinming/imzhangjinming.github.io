---
title: UNIX环境高级编程 第四章 -- 学习笔记
date: 2021-08-03 02:53:30
categories:
- notes
tags:
---

{% asset_img folder-146153_1280.png %}



APUE 第四章 文件和目录



<!-- more -->

# 第四章 文件和目录

## 4.2 stat fstat fstatat lstat 函数

```C
 #include<sys/stat.h>

int stat(const char *restrict pathname, struct stat *restrict buf);

int fstat(int fd, struct stat *buf);

int lstat(const char *restrict pathname,struct stat *restrict buf);

int fstatat(int fd, const char *restrict pathname, struct stat *restrict buf, int flag);
//成功返回 0； 出错返回 -1
```

`stat` 函数返回路径名代表文件的信息结构；

`fstat` 返回由文件描述符描述的文件的信息结构；

`lstat` 作用和 `stat` 差不多，只是如果文件是链接符号，`lstat` 会返回链接符号本身的信息结构而不是它所链接的文件的结构； 

`fstatat` 用来处理相对路径，其中 `flag` 参数可以是 `AT_SYMLINK_NOFOLLOW`(当文件是链接符号时，返回符号的信息) 或者 `AT_FDCWD` （返回符号链接所指向的实际文件的信息）



`buf` 指向一个结构体

```C
struct stat {
mode_t 			st_mode; 			/* file type & mode (permissions) */
ino_t			st_ino;				/* i-node number (serial number) */
dev_t 			st_dev;				/* device number (file system) */
dev_t			st_rdev;			/* device number for special files */
nlink_t			st_nlink;			/* number of links */
uid_t			st_uid;				/* user ID of owner */
gid_t			st_gid;				/* group ID of owner */
off_t			st_size;			/* size in bytes, for regular files */
struct timespec st_atim;			/* time of last access */
struct timespec st_mtim;			/* time of last modification */
struct timespec st_ctim;			/* time of last file status change */
blksize_t		st_blksize; 		/* best I/O block size */
blkcnt_t		st_blocks;			/* number of disk blocks allocated */
};
```



实际上在使用 `ls -l` 命令的时候就用到了 `stat` 函数



## 4.3 文件类型

* 普通文件 regular file
* 目录文件 directory file
* 块特殊文件 block special file
* 字符特殊文件 character special file
* FIFO 用于进程间通信
* 套接字 socket ，用于进程间的网络通信
* 符号链接 symbolic link ， 指向另一个文件



文件类型由 `stat` 结构体中的 `st_mode` 成员表示，可用以下宏确定文件类型

{% asset_img Snipaste_2021-08-01_09-35-26.png %}

[打印文件类型程序](https://github.com/imzhangjinming/APUE/blob/master/4/prt_filetype.c)



## 4.4 设置用户 ID 和 组 ID

每个进程有6个或者更多的关联ID，如下图

{% asset_img Snipaste_2021-08-01_10-01-49.png %}

通常，有效用户ID effective user ID = 实际用户ID real user ID

有效组ID effective group ID= 实际组ID real group ID



## 4.5 文件访问权限

访问权限保存在 `st_mode` 中，每个文件有9个访问权限位：

{% asset_img Snipaste_2021-08-01_10-10-30.png %}

`chmod` 命令可以用来改变这9个位



## 4.6 新文件和目录的所有权

新文件的**用户ID**设置为进程的**有效用户ID**

新问价的**组ID**有以下两种可能：

* 进程的**有效组ID**
* 所在**目录的组ID**



## 4.7 access 和 faccessat 函数

使用 `open` 函数打开一个文件时，内核以进程的**有效用户ID**和**有效组ID**执行访问权限测试。但有时内核也想使用**实际用户ID**和**实际组ID**执行访问权限测试

本节标题中的两个函数就是使用**实际用户ID**和**实际组ID**执行访问权限测试

```C
#include<unistd.h>

int access(const char *pathname, int mode);

int faccessat(int fd, const char *pathname, int mode, int flag);
// 成功返回 0； 出错返回 -1
```

[access函数的用法](https://github.com/imzhangjinming/APUE/blob/master/4/use_access.c)



## 4.8 umask 函数

umask 函数相当于为 **进程** 创建了一个模板，进程以后创建的文件的访问权限都根据这个模板来设置

```C
 #include<sys/stat.h>

mode_t umask(mode_t cmask);
```

`cmask`  里置1的位所代表的访问权限会被关闭，比如

`cmask = 000 111 111` （为了清晰，这里写的是比特位）

那么进程以后创建的所有文件的 group 和 other 的所有访问权限都会被关闭

[使用umask函数](https://github.com/imzhangjinming/APUE/blob/master/4/use_umask.c)



## 4.9 chmod 、 fchmod 和 fchmodat 函数

```C
 #include<sys/stat.h>

int chmod(const char *pathname, mode_t mode);

int fchmod(int fd, mode_t mode);

int fchmodat(int fd, const char *pathname, mode_t mode, int flag);
// 成功返回 0； 出错返回 -1
```

`mode` 可选下表中列出的值，或者它们的按位或

{% asset_img Snipaste_2021-08-01_13-06-43.png %}

[使用chmod函数](https://github.com/imzhangjinming/APUE/blob/master/4/use_chmod.c)



`chmod ` 函数在下列情况下**自动**清除两个权限位

* Solaris 等系统对普通文件的**粘着位**赋予了特殊含义。如果我们在这些系统上以非 root 用户设置普通文件的粘着位，粘着位会**自动关闭**
* 新创建文件的组ID可能不是调用进程所属的组。这种情况下，新文件的**设置组ID**位会被**自动关闭**



## 4.11 chown 、 fchown 、fchownat 和 lchown 函数

以上四个函数用来修改文件的用户ID和组ID

```C
#include<unistd.h>

int chown(const char *pathname, uid_t owner, gid_t group);

int fchown(int fd, uid_t owner,git_t group);

int fchownat(int fd, const char *pathname, uid_t owner, gid_t group, int flag);
//成功返回 0； 出错返回 -1
```

这四个函数的异同点和 `stat ` `fstat`  `fstatat`  `lstat`这四个函数的异同点相同



当 `_POSIX_CHOWN_RESTRICTED`  有效时，不能更改其他用户文件的用户ID。可以更改自己文件的组ID，但是只能改到自己**所属的组**



## 4.12 文件长度

`stat` 结构成员 `st_size` 表示以字节为单位的文件的长度。这个字段只对**普通文件**、**目录文件**和**符号链接**有意义



**文件中的空洞**

`du` 命令可以查看空洞文件实际占据的磁盘块数



## 4.13 文件截断

文件截断可以用来缩短文件。

如果想要截断文件，打开的时候要使用 `O_TRUNC` 标志

```C
#include<unistd.h>

int truncate(const char *pathname, off_t length);

int ftruncate(int fd, off_t length);
//成功返回 0； 出错返回 -1
```



## 4.14 文件系统

磁盘、分区和文件系统的关系：

{% asset_img Snipaste_2021-08-01_14-49-45.png %}



将上图中的 i-nodes 和 data blocks 放大来看：

{% asset_img Snipaste_2021-08-01_14-52-49.png %}

* 图中，有两个目录项指向同一个 i-node 。每个 i-node 有一个**链接计数**，表示指向它的目录项的个数，只有当**链接计数为 0** 时才能够删除该文件（和智能指针管理动态内存的方法类似）。`stat` 结构中， 链接计数保存在 `st_nlink` 成员中。这种链接类型称为**硬链接**

* 另外一种链接是**符号链接（symbolic link）**。符号链接内只保存了它链接的文件的名字

* i-node 包含了文件有关的所有信息：文件类型、文件访问权限位、文件长度和指向文件数据块的指针等。

  **目录项**中存放 **i-node 编号**和**文件名**



介绍完了文件的链接计数，那么**目录**是如何进行链接计数的呢？ 假设我们运行下面的命令创建了一个新的文件夹 `testdir`

```shell
$ mkdir testdir
```

关于它的链接计数形式请看下图

{% asset_img Snipaste_2021-08-01_15-10-21.png %}



## 4.15 link 、 linkat 、unlink 、unlinkat 和 remove 函数

创建一个指向现有文件的链接的方法是使用 link 或 linkat 函数：

```C
 #include<unistd.h>

int link(const char *existingpath, const char *newpath);

int linkat(int efd, const char *existingpath, int nfd, const char *newpath, int flag);
//成功返回 0； 出错返回 -1
```

为了删除一个现有的目录项，可以使用 unlink 函数：

```C
#include<unistd.h>

int unlink(const char *pathname);
int unlink(int fd, const char *pathname, int flag);
//成功返回 0；出错返回 -1
```



[使用 unlink 函数](https://github.com/imzhangjinming/APUE/blob/master/4/use_unlink.c)



**remove**函数可以解除对一个**文件**或**目录**的链接

```C
 #include<stdio.h>

int remove(const char *pathname);
//成功返回 0； 出错返回 -1
```



## 4.16 rename 和 renameat 函数

从命名可以看出，这俩函数是用来重命名的

```C
 #include<stdio.h>

int rename(const char *oldname, const char *newname);

int renameat(int oldfd, const char *oldname, int newfd, const char *newname);
//成功返回 0； 出错返回 -1
```



## 4.17  符号链接

主要要区分符号链接和**硬链接**的区别

* 符号链接是对一个文件的间接指针

* 硬链接直接指向文件的 i-node



使用符号链接的目的是为了**避开硬链接的一些限制**

* 硬链接通常要求链接和文件位于**同一文件系统中**
* 只有 root 用户才能创建指向**目录**的硬链接

符号链接则**没有**以上两种限制



在使用函数的时候要注意其是否处理符号链接。如果函数不处理符号链接，那么它将处理链接本身**而不是**链接指向的文件

下图展示了一些函数对符号链接的支持情况

{% asset_img Snipaste_2021-08-01_17-36-05.png %}



`ftw` **函数可以遍历文件结构，打印遇到的每个路径名**



## 4.18 创建和读取符号链接

使用以下俩函数创建符号链接：

```C
#include<unistd.h>

int symlink(const char *actualpath, const char *sympath);
int symlinkat(const char *actualpath, int fd, const char *sympath);
//成功返回 0； 出错返回 -1
```



我们知道 `open` 函数会跟随符号链接打开链接指向的文件，那么怎么打开链接自身呢？

使用下面这俩函数：

```C
#include<unistd.h>

ssize_t readlink(const char *restrict pathname, char *restrict buf, size_t bufsize);

ssize_t readlinkat(int fd, const char *restrict pathname, char *restrict buf, size_t bufsize);
//成功返回读取的字节数； 出错返回 -1
```



## 4.19 文件的时间

UNIX系统为每个文件维护三个时间字段，如下图：

{% asset_img Snipaste_2021-08-01_17-57-17.png %}

为什么有了 `st_mtim` 文件数据最后修改的时间，还需要 `st_ctim` i-node最后被修改的时间呢？

因为 文件数据 和 i-node 里的信息是 **完全分开存放的**。



## 4.20 futimens 、utimensat 和 utimes 函数

这仨函数用来更改文件的访问和修改时间

```C
#include<sys/stat.h>

int futimens(int fd, const struct timespec times[2]);
int utimensat(int fd, const char *path, const struct timespec times[2], int flag);
//以上两个函数可以指定精确到纳秒级的时间戳

#include<sys/time.h>

int utimes(const char *pathname, const struct timeval times[2]);
//成功返回 0； 出错返回 -1
```



`timeval`   是结构体，它的结构如下

```C
struct timeval{
    time_t tv_sec; /* seconds */
    long tv_usec;  /* microseconds */
};
```

`times` 是个数组，第一个元素指定访问时间 `st_atim` ，第二个元素指定修改时间 `st_mtim` 



## 4.21 mkdir 、mkdirat 和 rmdir 函数

mkdir 、mkdirat 创建目录，rmdir 删除目录

```C
#include<sys/stat.h>

int mkdir(const char *pathname, mode_t mode);

int mkdirat(int fd, const char *pathname, mode_t mode);
//成功返回 0； 出错返回 -1
```

创建目录的时候需要注意，对于目录通常至少要设置**执行权限位**，以允许访问该目录中的文件名

```C
#include<unistd.h>

int rmdir(const char *pathname);
//成功返回 0； 出错返回 -1
```

`rmdir` 可以删除一个**空目录**



## 4.22 读目录

```C
#include <dirent.h>
DIR *opendir(const char *pathname);
DIR *fdopendir(int fd);
//Both return: pointer if OK, NULL on error
    
struct dirent *readdir(DIR *dp);
//Returns: pointer if OK, NULL at end of directory or error
    
void rewinddir(DIR *dp);
int closedir(DIR *dp);
//Returns: 0 if OK, −1 on error

//以下两个函数不是基本 POSIX.1 标准的组成部分，而是 SUS 的 XSI 扩展
long telldir(DIR *dp);
//Returns: current location in directory associated with dp

void seekdir(DIR *dp, long loc);
```

`dirent` 结构体至少包含 `ino_t d_ino`  `char d_name[]`  这两个成员，前者是 i-node 编号，后者是文件名



[ myftw 函数 ](https://github.com/imzhangjinming/APUE/blob/master/4/myftw.c)



## 4.23 chdir 、fchdir 和 getcwd 函数

进程调用chdir 和 fchdir 函数可以更改进程当前的工作目录

```C
#include<unistd.h>

int chdir(const char *pathname);

int fchdir(int fd);
//成功返回 0； 出错返回 -1
```

`pathname`  和 `fd` 都用来指定新的工作目录

[使用 chdir 函数](https://github.com/imzhangjinming/APUE/blob/master/4/use_chdir.c)



## 4.25 文件访问权限小结

{% asset_img Snipaste_2021-08-01_19-19-38.png %}

**S_IRWXU = S_IRUSR | S_IWUSR | S_IXUSR**
**S_IRWXG = S_IRGRP | S_IWGRP | S_IXGRP**
**S_IRWXO = S_IROTH | S_IWOTH | S_IXOTH**



# 课后习题

## 4.6 编写一个类似 cp(1) 的程序，它复制包含空洞的文件，但不将 字节 0 写到输出文件中

[我的解答](https://github.com/imzhangjinming/APUE/blob/master/4/exercise/mycp.c)

## 4.11 对 myftw 函数进行更改：每次遇到一个目录就用其调用 chdir，这样每次调用 lstat 时就可以使用文件名而非路径名，处理完所有的目录项后执行 chdir("..")。比较这两个版本的程序的运行时间

[我的解答](https://github.com/imzhangjinming/APUE/blob/master/4/exercise/myftw.c)

