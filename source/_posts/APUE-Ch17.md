---
title: UNIX环境高级编程 第十七章 高级进程间通信
date: 2021-08-20 20:15:59
categories:
- notes
tags:
---


# 第十七章 高级进程间通信

## 17.2 UNIX域套接字

用于在**同一台**计算机上运行的进程间通信。像是套接字和管道的组合

使用 `socketpair` 函数创建一对无命名的、相互连接的UNIX域套接字

```C
#include <sys/socket.h>

int socketpair(int domain, int type, int protocol, int sockfd[2]);
//Returns: 0 if OK, −1 on error
```

{% asset_img Snipaste_2021-08-18_14-45-08.png %}

一对相互连接的UNIX域套接字可以起到全双工管道的作用，两端都对**读和写**开放



**命名UNIX域套接字**

就像网络域套接字一样，可以把一个地址绑定到UNIX域套接字上。

UNIX域套接字地址由 `sockaddr_un` 结构表示

Linux 和 Solaris 实现

```C
struct sockaddr_un {
	sa_family_t sun_family;		/* AF_UNIX */
	char 		sun_path[108];	/* pathname */
};
```

FreeBSD 和 Mac OS 实现

```C
struct sockaddr_un {
	unsigned char 	sun_len; 		/* sockaddr length */
	sa_family_t 	sun_family;		/* AF_UNIX */
	char 			sun_path[104];	/* pathname */
};
```

绑定地址的时候，系统会使用 `sun_path` 指定的路径名创建一个文件，如果该文件已经存在，则 `bind` 请求失败

还要注意的是关闭套接字的时候并**不自动删除**该文件，所以要在程序退出前对该文件执行**解除链接** 的操作



[将地址绑定到UNIX域套接字](https://github.com/imzhangjinming/APUE/blob/master/17/bind_addr_unix_socket.c)



## 17.3 唯一连接

开发3个函数，在同一台计算机上的两个无关进程之间建立**唯一**连接

```C
#include "apue.h"

int serv_listen(const char *name);
//Returns: file descriptor to listen on if OK, negative value on error

int serv_accept(int listenfd, uid_t *uidptr);
//Returns: new file descriptor if OK, negative value on error

int cli_conn(const char *name);
//Returns: file descriptor if OK, negative value on error
```

`serv_listen` 函数的 `name` 参数声明了服务器将在这个文件上监听客户的连接请求，此函数的返回值是用于接收客户连接请求的服务器UNIX域套接字

服务器进程使用 `serv_accept` 函数等待客户进程连接请求的到达。当客户请求到达时，服务器自动创建一个新的UNIX域套接字并将它与客户端套接字相连

客户进程使用 `cli_conn` 函数连接至服务器进程，`name` 参数就是 调用 `serv_listen` 是指定的 `name` ，此函数返回值是连接到服务器进程的文件描述符

三个函数的实现：

[serv_listen](https://github.com/imzhangjinming/APUE/blob/master/17/serv_listen.c)

[serv_accept](https://github.com/imzhangjinming/APUE/blob/master/17/serv_accept.c)

[cli_conn](https://github.com/imzhangjinming/APUE/blob/master/17/cli_conn.c)



