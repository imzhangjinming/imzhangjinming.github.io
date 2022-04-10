---
title: UNIX环境高级编程 第六章 -- 学习笔记
date: 2021-08-04 23:06:37
categories:
- notes
tags:
---

{% asset_img analytics-3088958_1920.jpg %}

APUE 第六章 系统数据文件和信息



<!-- more -->

# 第六章 系统数据文件和信息

## 6.2 口令文件

口令文件指的是 /etc/passwd 文件，该文件中包含了下图中所示的各个字段

{% asset_img Snipaste_2021-08-04_08-07-17.png %}

这些字段包含在 `<pwd.h>` 中定义的 `passwd` 结构中

那么该怎么获得口令文件中的项呢？

使用下面这俩函数

```C
#include <pwd.h>

struct passwd *getpwuid(uid_t uid);

struct passwd *getpwnam(const char *name);
//Both return: pointer if OK, NULL on error
```

这两个函数都返回一个指向 `passwd` 结构体的指针，结构体里包含了图 6.1 所示的字段



如果想查看**整个口令文件**，使用下面的三个函数

```C
#include <pwd.h>

struct passwd *getpwent(void);
//Returns: pointer if OK, NULL on error or end of file

void setpwent(void);
void endpwent(void);
```

每次调用 `getpwent` 都返回口令文件中的下一个记录项，第一次调用时会打开所有它要用到的文件。  `setpwent`  rewinds 它使用的文件。 `endpwent`  关闭它使用的那些文件。



[使用 getpwent、setpwent 和 endpwent  实现 getpwnam](https://github.com/imzhangjinming/APUE/blob/master/6/mygetpwname.c)



## 6.3 阴影口令 shadow passwords

这个文件是专门用来放置 **加密后的密码** 的

shadow passwords 中的字段如下图所示

{% asset_img Snipaste_2021-08-04_08-49-44.png %}

这其中只有 **用户名** 和 **加密后的口令** 是必需的

那么如何访问阴影口令文件呢？ 和口令文件类似，有如下函数

```C
#include <shadow.h>

struct spwd *getspnam(const char *name);

struct spwd *getspent(void);
//Both return: pointer if OK, NULL on error

void setspent(void);
void endspent(void);
```



## 6.4 组文件 group file

组文件中包含了下图中的各个字段

{% asset_img Snipaste_2021-08-04_08-54-01.png %}

这些字段包含在 `<grp.h>` 中定义的 `group` 结构体中



类似地，可以使用下面的函数来查看组名或者 group ID

```C
#include <grp.h>

struct group *getgrgid(gid_t gid);

struct group *getgrnam(const char *name);
//Both return: pointer if OK, NULL on error
```



如果要搜索整个组文件，使用下面三个函数

```C
#include <grp.h>

struct group *getgrent(void);
//Returns: pointer if OK, NULL on error or end of file

void setgrent(void);
void endgrent(void);
```

每次调用 `getgrent` 都返回口令文件中的下一个记录项，第一次调用时会打开所有它要用到的文件。  `setgrent`  rewinds 它使用的文件。 `endgrent`  关闭它使用的那些文件。



## 6.5 附属组 ID

为了获取和设置附属组 ID，提供了以下函数

```C
#include <unistd.h>

int getgroups(int gidsetsize, gid_t grouplist[]);
//Returns: number of supplementary group IDs if OK, −1 on error


#include <grp.h>
/* on Linux */
#include <unistd.h> /* on FreeBSD, Mac OS X, and Solaris */

int setgroups(int ngroups, const gid_t grouplist[]);


#include <grp.h>
/* on Linux and Solaris */
#include <unistd.h> /* on FreeBSD and Mac OS X */

int initgroups(const char *username, gid_t basegid);
//Both return: 0 if OK, −1 on error
```

``getgroups` 将用户所属的附属组名写到数组 `grouplist` 中，写入的个数由 `gidsetsize` 指定。实际写入的个数由函数返回。



## 6.6 实现区别

各种平台存储用户和组信息的方式：

{% asset_img Snipaste_2021-08-04_09-09-26.png %}



## 6.7 其他数据文件

除了 口令文件 和 组文件，UNIX系统还使用很多其他文件。

一般来说，对于每个数据文件至少有三个函数

* get 函数： 读取下一个记录。 通常返回一个指向结构体的指针。此类指针通常指向一个静态存储类的结构，如果要保存返回的内容，则应该保存整个结构体，而非指针，因为返回指针指向的结构体在下一次调用 get  函数的时候会被重写
* set 函数：
* end 函数：关闭相应的数据文件



UNIX系统其他数据文件的相关信息：

{% asset_img Snipaste_2021-08-04_09-16-28.png %}



## 6.8 登录账户记录

大多数UNIX 系统提供下列两个数据文件： utmp 和 wtmp 

每次写入这两个文件中的是下面的这个结构

```C
struct utmp {
	char ut_line[8]; 	/* tty line: "ttyh0", "ttyd0", "ttyp0", ... */
	char ut_name[8]; 	/* login name */
	long ut_time;		/* seconds since Epoch */
};
```



## 6.9 系统标识

POSIX.1 定义了 uname 函数，它返回与主机和操作系统有关的信息

```C
#include <sys/utsname.h>

int uname(struct utsname *name);
//Returns: non-negative value if OK, −1 on error
```

其中 `utsname` 结构体如下

```C
struct utsname {
	char sysname[];		/* name of the operating system */
	char nodename[];	/* name of this node */
	char release[];		/* current release of operating system */
	char version[];		/* current version of this release */
	char machine[];		/* name of hardware type */
};
```

`uname` 负责填写 `name` 参数指向的结构体



## 6.10 时间和日期

UNIX内核提供的基本时间服务是计算自 **协调世界时** （Coordinated Universal Time,UTC）公元1970年1月1日 00:00:00 以来经过的 **秒数** ，数据类型是 `time_t` 

`time` 函数返回当前日期和时间

```C
#include <time.h>

time_t time(time_t *calptr);
//Returns: value of time if OK, −1 on error
```



POSIX.1 增加了多个系统时钟，用 `clockid_t` 类型的变量标识，如下图

{% asset_img Snipaste_2021-08-04_09-32-13.png %}



使用 `clock_gettime` 函数获取指定时钟的时间

```C
#include <sys/time.h>

int clock_gettime(clockid_t clock_id, struct timespec *tsp);
//Returns: 0 if OK, −1 on error
```

 要对特定的时钟设置时间，使用

```C
#include <sys/time.h>

int clock_settime(clockid_t clock_id, const struct timespec *tsp);
//Returns: 0 if OK, −1 on error
```

注意，修改时钟**需要特权**，而且有些时钟是**不能修改**的



注意，获得的 time_t 类型的整型值是一个很大的数字，我们不能从它直接得到当前的日期和时间。这时候就需要各种转换函数 了

`localtime`  和 `gmtime` 两个函数将转换后的日期时间放在 `tm` 结构体中

```C
struct tm {/* a broken-down time */
	int tm_sec;		/* seconds after the minute: [0 - 60] */
	int tm_min;		/* minutes after the hour: [0 - 59] */
	int tm_hour; 	/* hours after midnight: [0 - 23] */
	int tm_mday; 	/* day of the month: [1 - 31] */
	int tm_mon;		/* months since January: [0 - 11] */
	int tm_year; 	/* years since 1900 */
	int tm_wday; 	/* days since Sunday: [0 - 6] */
	int tm_yday; 	/* days since January 1: [0 - 365] */
	int tm_isdst; 	/* daylight saving time flag: <0, 0, >0 */
};
```



下图反映了各个时间函数之间的关系

{% asset_img Snipaste_2021-08-04_09-42-14.png %}



```C
#include <time.h>

struct tm *gmtime(const time_t *calptr);

struct tm *localtime(const time_t *calptr);
//Both return: pointer to broken-down time, NULL on error
```

这俩函数的区别在于： `gmtime` 把日历时间（很大的那个秒数） 转换为 协调统一时间的年月日分秒；而 `localtime` 把日历时间转换为当地的年月日分秒（考虑时区和夏令时）



函数 `gmtime`  把本地时间的年月日等作为参数，将其变成 `time_t` 值

```C
#include <time.h>

time_t mktime(struct tm *tmptr);
//Returns: calendar time if OK, −1 on error
```



下面这两个函数用来以不同的格式打印时间

```C
#include <time.h>

size_t strftime(char *restrict buf, size_t maxsize,const char *restrict format,const struct tm *restrict tmptr);

size_t strftime_l(char *restrict buf, size_t maxsize,const char *restrict format,const struct tm *restrict tmptr, locale_t locale);
//Both return: number of characters stored in array if room, 0 otherwise
```

`strftime_l` 函数除了可以用 `locale` 参数指定时区外与 `strftime` 函数没有区别

 `format` 参数与 `printf` 函数中的  `format` 参数类似，它控制打印的格式。自然地，  `format` 参数中也会包含转换说明

{% asset_img 227.png %}

