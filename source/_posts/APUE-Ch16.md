---
title: UNIX环境高级编程 第十六章 网络套接字
date: 2021-08-17 16:43:47
categories:
- notes
tags:
---


{% asset_img network-3139214_1920.jpg %}

APUE 第十六章 socket套接字



<!-- more -->

# 第十六章 网络IPC：套接字 socket

套接字用来进行不同计算机上的进程相互通信

## 16.2 套接字描述符 

应用程序使用套接字描述符访问套接字。在UNIX系统中，套接字描述符被看作一种文件描述符。

为了**创建**一个套接字，调用 `socket` 函数

```C
#include <sys/socket.h>

int socket(int domain, int type, int protocol);
//Returns: file (socket) descriptor if OK, −1 on error
```

`domain` 参数指定地址族 address family ，不同地址族的地址格式不同，表示地址族的常数以 `AF_` 开头

{% asset_img Snipaste_2021-08-15_14-33-47.png %}

`type` 参数指定套接字的类型，进一步确定通信特征

{% asset_img Snipaste_2021-08-15_14-41-59.png %}

`protocol` 通常是 0 ，表示为给定的域和套接字选择默认协议。

`AF_INET` 通信域，`SOCK_STREAM` 的默认协议是 传输控制协议 （Transmission Control Protocol，TCP）

`AF_INET` 通信域，`SOCK_DGRAM` 的默认协议是 用户数据报协议 （User Datagram Protocol，UDP）

下图是一些协议

{% asset_img Snipaste_2021-08-15_14-45-34.png %}

`socket` 函数返回套接字描述符。本质上它是一个文件描述符，所以很多以文件描述符为参数的函数同样可以使用套接字描述符，下图定义了一些函数以套接字描述符为参数调用时的行为

{% asset_img Snipaste_2021-08-15_14-50-56.png %}

套接字通信是**双向**的，用 `shutdown` 函数来禁止一个套接字的 I/O

```C
#include <sys/socket.h>

int shutdown(int sockfd, int how);
//Returns: 0 if OK, −1 on error
```

`how` 可以是  `SHUT_RD`、`SHUT_WR` 和 `SHUT_RDWR`，分别关闭 读、写 和 读写



## 16.3 寻址  addressing

如何准确确定目标通信地址 ？ 计算机网络地址 + 端口号



### 16.3.1 字节序

字节序是一个处理器架构特性，用于指示大数据类型（如整型）内部的字节如何排序

{% asset_img Snipaste_2021-08-15_16-08-27.png %}

MSB ： Most Significant Byte 最高有效字节

LSB： Least Significant Byte 最低有效字节

由上图可见，大端（big-endian） 字节由MSB向 LSB排列，小端则相反

TCP/IP 使用**大端字节序**。

所以应用程序有时需要在处理器字节序和网络字节序之间实施转换。

对于TCP/IP应用程序，有4个函数能完成此转换操作

```C
#include <arpa/inet.h>

uint32_t htonl(uint32_t hostint32);
//Returns: 32-bit integer in network byte order

uint16_t htons(uint16_t hostint16);
//Returns: 16-bit integer in network byte order

uint32_t ntohl(uint32_t netint32);
//Returns: 32-bit integer in host byte order

uint16_t ntohs(uint16_t netint16);
//Returns: 16-bit integer in host byte order
```



### 16.3.2  地址格式

不同的通信域有不同的地址格式，为了让不同格式的地址能够传入套接字，地址会被强制转换成 一个通用的结构 `sockaddr`

```C
struct sockaddr {
	sa_family_t 	sa_family;	/* address family */
	char 			sa_data[];	/* variable-length address */
	...
};
```



在 IPv4 因特网域（`AF_INET`）中，套接字地址用结构 `sockaddr_in` 表示

```C
struct in_addr {
	in_addr_t 	s_addr;	/* IPv4 address */
};
```

```C
struct sockaddr_in {
	sa_family_t 	sin_family;	/* address family */
	in_port_t 		sin_port;	/* port number */
	struct in_addr 	sin_addr;	/* IPv4 address */
};
```



在 IPv6 因特网域（`AF_INET6`）中，套接字地址用结构 `sockaddr_in6` 表示

```C
struct in6_addr {
	uint8_t 	s6_addr[16];	/* IPv6 address */
};
```

```C
struct sockaddr_in6 {
	sa_family_t 	sin6_family;	/* address family */
	in_port_t 		sin6_port;		/* port number */
	uint32_t 		sin6_flowinfo;	/* traffic class and flow info */
	struct in6_addr sin6_addr;		/* IPv6 address */
	uint32_t 		sin6_scope_id;	/* set of interfaces for scope */
};
```



有时为了打印出人类易于观察的地址，需要在二进制地址格式和点分十进制字符之间进行转换，以下两个函数负责这个操作

```C
#include <arpa/inet.h>

const char *inet_ntop(int domain, const void *restrict addr,char *restrict str, socklen_t size);
//Returns: pointer to address string on success, NULL on error

int inet_pton(int domain, const char *restrict str,void *restrict addr);
//Returns: 1 on success, 0 if the format is invalid, or −1 on error
```

`inet_ntop` 将二进制地址转换成文本字符串格式，`size` 指定缓冲区 `str` 的大小。`INET_ADDRSTRLEN` 指定了足够容纳 IPv4 地址的大小，`INET6_ADDRSTRLEN` 指定了足够容纳 IPv6 地址的大小



`inet_pton` 将文本字符串格式转换为二进制地址

`domain` 参数 可选 `AF_INET` 和 `AF_INET6`



### 16.3.3 地址查询 address lookup

`gethostent` 函数获得给定计算机系统的主机信息

```C
#include <netdb.h>

struct hostent *gethostent(void);
//Returns: pointer if OK, NULL on error

void sethostent(int stayopen);

void endhostent(void);
```

`hostent` 结构如下

```C
struct hostent {
	char   *h_name; 		/* name of host */
	char  **h_aliases; 		/* pointer to alternate host name array */
	int 	h_addrtype; 	/* address type */
	int 	h_length; 		/* length in bytes of address */
	char  **h_addr_list; 	/* pointer to array of network addresses */
	...
};
```



下面这些函数获得网络名字和网络编号

```C
#include <netdb.h>

struct netent *getnetbyaddr(uint32_t net, int type);

struct netent *getnetbyname(const char *name);

struct netent *getnetent(void);
//All return: pointer if OK, NULL on error

void setnetent(int stayopen);

void endnetent(void);
```

`netent` 结构

```C
struct netent {
	char 	   *n_name;		/* network name */
	char 	  **n_aliases;	/* alternate network name array pointer */
	int 		n_addrtype;	/* address type */
	uint32_t 	n_net;		/* network number */
	...
};
```



协议名 和 协议编号 之间的映射

```C
#include <netdb.h>

struct protoent *getprotobyname(const char *name);

struct protoent *getprotobynumber(int proto);

struct protoent *getprotoent(void);
//All return: pointer if OK, NULL on error

void setprotoent(int stayopen);

void endprotoent(void);
```

`protoent` 结构体

```C
struct protoent {
	char   *p_name;		/* protocol name */
	char  **p_aliases; 	/* pointer to alternate protocol name array */
	int 	p_proto;	/* protocol number */
	...
};
```



服务和端口号之间的映射

```C
#include <netdb.h>

struct servent *getservbyname(const char *name, const char *proto);

struct servent *getservbyport(int port, const char *proto);

struct servent *getservent(void);
//All return: pointer if OK, NULL on error

void setservent(int stayopen);

void endservent(void);
```

`servent` 结构

```C
struct servent {
	char     *s_name;		/* service name */
	char 	**s_aliases; 	/* pointer to alternate service name array */
	int 	  s_port; 		/* port number */
	char 	 *s_proto;		/* name of protocol */
	...
};
```



`getaddrinfo` 函数允许将一个主机名和一个服务名映射到一个地址

```C
#include <sys/socket.h>
#include <netdb.h>

int getaddrinfo(const char *restrict host,const char *restrict service,const struct addrinfo *restrict hint,struct addrinfo **restrict res);
//Returns: 0 if OK, nonzero error code on error

void freeaddrinfo(struct addrinfo *ai);
```

`addrinfo` 结构

```C
struct addrinfo {
	int 				ai_flags;		/* customize behavior */
	int 				ai_family;		/* address family */
	int 				ai_socktype;	/* socket type */
	int 				ai_protocol; 	/* protocol */
	socklen_t 			ai_addrlen;		/* length in bytes of address */
	struct sockaddr    *ai_addr; 		/* address */
	char 			   *ai_canonname; 	/* canonical name of host */
	struct addrinfo    *ai_next; 		/* next in list */
	...
};
```

可以提供一个 可选的 `hint` 来选择符合特定条件的地址。`hint` 相当于一个过滤器，可以过滤 `ai_family`、`ai_flags`、`ai_protocol` 和 `ai_socktype` 字段，但是其它的字段必须设置为 0 。

`ai_flags` 可选值如下

{% asset_img Snipaste_2021-08-15_22-26-44.png %}

如果 `getaddrinfo` 失败，要调用 `gai_strerror` 将返回的错误码转换成错误消息

```C
#include <netdb.h>

const char *gai_strerror(int error);
//Returns: a pointer to a string describing the error
```



`getnameinfo` 函数将一个地址转换成一个主机名和一个服务名

```C
#include <sys/socket.h>
#include <netdb.h>

int getnameinfo(const struct sockaddr *restrict addr, socklen_t alen,char *restrict host, socklen_t hostlen,char *restrict service, socklen_t servlen, int flags);
//Returns: 0 if OK, nonzero on error
```

`flags` 参数指定了控制翻译的方式

{% asset_img Snipaste_2021-08-15_22-30-07.png %}



[使用 getaddrinfo 函数的例子](https://github.com/imzhangjinming/APUE/blob/master/16/use_getaddrinfo.c)



### 16.3.4 将套接字与地址关联

`bind` 函数关联套接字和地址

```C
#include <sys/socket.h>

int bind(int sockfd, const struct sockaddr *addr, socklen_t len);
//Returns: 0 if OK, −1 on error
```



`getsockname` 函数发现绑定到套接字上的地址

```C
#include <sys/socket.h>

int getsockname(int sockfd, struct sockaddr *restrict addr,socklen_t *restrict alenp);
//Returns: 0 if OK, −1 on error
```

调用前，`alenp` 设置为一个指向整数的指针，该整数指定缓冲区 `addr` 的长度



如果 `socket` 已经和对方连接，可以使用  `getpeername` 函数找到对方的地址

```C
#include <sys/socket.h>

int getpeername(int sockfd, struct sockaddr *restrict addr,socklen_t *restrict alenp);
//Returns: 0 if OK, −1 on error
```



## 16.4 建立连接

`connect` 负责建立连接

```C
#include <sys/socket.h>

int connect(int sockfd, const struct sockaddr *addr, socklen_t len);
//Returns: 0 if OK, −1 on error
```

由于一些原因，连接可能会失败，应用程序必须能够处理 `connect` 返回的错误

[处理 connect 错误的例子](https://github.com/imzhangjinming/APUE/blob/master/16/connect_retry.c)

[可移植的处理 connect 错误的例子](https://github.com/imzhangjinming/APUE/blob/master/16/portable_connect_retry.c)



服务器 调用 `listen` 函数表明可以接受连接请求

```C
#include <sys/socket.h>

int listen(int sockfd, int backlog);
//Returns: 0 if OK, −1 on error
```



一旦调用了 `listen` ，服务器就能接收连接请求。使用 `accept` 函数获得连接请求并建立连接

```C
#include <sys/socket.h>

int accept(int sockfd, struct sockaddr *restrict addr,socklen_t *restrict len);
//Returns: file (socket) descriptor if OK, −1 on error
```

此函数返回值是一个连接到 调用 `connect` 的客户端的套接字描述符。

如果不关心客户端标识，后两个参数可以设置为 `NULL`；否则应该为 `addr` 分配足够大的缓冲区，并用 `len` 指出此缓冲区的长度



如果没有连接请求 ，`accept` 会阻塞到请求到；除非 `sockfd` 设置为 非阻塞的 ，这种情况下如果没有连接请求，`accept` 会返回 `-1` 并将 `errno` 设置为 `EAGAIN`



[分配并初始化服务器端套接字](https://github.com/imzhangjinming/APUE/blob/master/16/init_server_socket.c)



## 16.5 数据传输

用于通过套接字发送数据的函数

```C
#include <sys/socket.h>

ssize_t send(int sockfd, const void *buf, size_t nbytes, int flags);
//Returns: number of bytes sent if OK, −1 on error
```

除了第四个参数，这个函数和 `write` 函数很类似，`flags` 常见选项如下

{% asset_img Snipaste_2021-08-15_21-54-59.png %}



```C
#include <sys/socket.h>

ssize_t sendto(int sockfd, const void *buf, size_t nbytes, int flags,const struct sockaddr *destaddr, socklen_t destlen);
//Returns: number of bytes sent if OK, −1 on error
```

`sendto` 与 `send` 的不同之处在于 `sendto` 可以在**无连接**的套接字上指定一个目标地址



```C
#include <sys/socket.h>

ssize_t sendmsg(int sockfd, const struct msghdr *msg, int flags);
//Returns: number of bytes sent if OK, −1 on error
```

`msghdr` 结构

```C
struct msghdr {
	void 		   *msg_name;		/* optional address */
	socklen_t 		msg_namelen;	/* address size in bytes */
	struct iovec   *msg_iov;		/* array of I/O buffers */
	int 			msg_iovlen;		/* number of elements in array */
	void 		   *msg_control;	/* ancillary data */
	socklen_t 		msg_controllen; /* number of ancillary bytes */
	int 			msg_flags;		/* flags for received message */
	...
};
```

通过 `sendmsg` 和 `msghdr` 结构可以在发送数据时指定多重缓冲区（与 `writev` 函数类似）



接下来是接受数据的函数

```C
#include <sys/socket.h>

ssize_t recv(int sockfd, void *buf, size_t nbytes, int flags);
//Returns: length of message in bytes,0 if no messages are available and peer has done an orderly shutdown,or −1 on error
```

`flags` 选项

{% asset_img Snipaste_2021-08-15_22-04-10.png %}



`recvfrom` 函数可以在接收数据的时候顺便保存发送端的信息

```C
#include <sys/socket.h>

ssize_t recvfrom(int sockfd, void *restrict buf, size_t len, int flags,struct sockaddr *restrict addr,socklen_t *restrict addrlen);
//Returns: length of message in bytes,0 if no messages are available and peer has done an orderly shutdown,or −1 on error
```



`recvmsg` 可以用 `msghdr` 结构指定多重缓冲区以存放接收到的数据

```C
#include <sys/socket.h>

ssize_t recvmsg(int sockfd, struct msghdr *msg, int flags);
//Returns: length of message in bytes,0 if no messages are available and peer has done an orderly shutdown,or −1 on error
```



[面向 连接的 客户端](https://github.com/imzhangjinming/APUE/blob/master/16/connect_oriented_client.c)

[面向 连接的 服务器端](https://github.com/imzhangjinming/APUE/blob/master/16/connect_oriented_server.c)



[使用 UDP 协议的客户端](https://github.com/imzhangjinming/APUE/blob/master/16/UDP_client.c)

[使用 UDP 协议的服务器端](https://github.com/imzhangjinming/APUE/blob/master/16/UDP_server.c)



## 16.6 套接字选项

```C
#include <sys/socket.h>

int setsockopt(int sockfd, int level, int option, const void *val,socklen_t len);
//Returns: 0 if OK, −1 on error
```

`option` 可选项：

{% asset_img Snipaste_2021-08-17_08-21-28.png %}



```C
#include <sys/socket.h>

int getsockopt(int sockfd, int level, int option, void *restrict val,socklen_t *restrict lenp);
//Returns: 0 if OK, −1 on error
```



## 16.7 带外数据 Out-of-Band Data

与普通数据相比，带外数据允许更高优先级的数据传输。

TCP 支持带外数据，UDP不支持。TCP 将带外数据称为 紧急数据（urgent data）



# 课后习题

## 1.写一个程序判断所使用系统的字节序

[我的解答](https://github.com/imzhangjinming/APUE/blob/master/16/exercise/big_or_little_endian.c)

 树莓派是小端

## 2.写一个程序，在至少两种不同的平台上打印出所支持套接字的 stat 结构成员，并且描述这些结果的不同之处

[我的解答](https://github.com/imzhangjinming/APUE/blob/master/16/exercise/print_socket_stat.c)



## 3.[此程序]16-17 只在一个端点上提供了服务。修改这个程序，同时支持多个端点上的服务

[解答](https://github.com/imzhangjinming/APUE/blob/master/16/exercise/multi-end_server.c)



