---
title: UNIX环境高级编程 第五章 -- 学习笔记
date: 2021-08-03 23:47:02
categories:
- notes
tags:
---

{% asset_img computer-1295125_1280.png %}

APUE 第五章 标准I/O库

<!-- more -->

# 第五章 标准I/O库



## 5.2 流和FILE对象

对于标准I/O库，它们的操作是围绕 **流**（stream）展开的。当用标准I/O库打开或者创建一个文件时，我们已使一个流与一个文件相关联。

流有**单字节流**和**多字节**流之分，决定一个流的类型的操作称为**流的定向**（orientation）。

`freopen ` 函数清除一个流的定向；`fwide` 函数设置一个流的定向

```C
#include<stdio.h>
#include<wchar.h>

int fwide(FILE *fp, int mode);
//流是宽定向的，返回正值；是字节定向的，返回负值； 未定向的，返回0
```

根据 `mode` 的不同值，`fwide` 函数有不同的行为

*  `mode` 为正，函数试图将流定向为宽字节

*  `mode` 为负，函数视图将流定向为单字节

*  `mode` 为0，函数什么也不做，并返回标识该流定向的值



 `FILE` 结构体是标准I/O库里最重要的一个结构体，指向它的指针  `FILE *` 称为**文件指针**



## 5.3 标准输入、标准输出和标准错误

标题里的三个流是为每个进程预定义的，分别通过文件指针 `stdin` 、`stdout` 、和 `stderr` 引用，它们定义在 `<stdio.h>` 头文件中



## 5.4 缓冲

缓冲一般都是为了提高效率，标准I/O的缓冲也一样，是为了减少 `read `  和 `write` 的调用次数

标准I/O提供三种缓冲

* 全缓冲。缓冲区**全部填满**后才**冲洗**（flush）。冲洗就是将缓冲区的内容写到磁盘
* 行缓冲。输入输出中出现换行符时执行冲洗。当然，这个行不能太长，要是缓冲区全部填满了还没有出现换行符，也是要执行冲洗的。
* 不带缓冲。`stderr` 通常是不带缓冲的。



不同的流的缓冲类型一般是下面这样的：

* 标准错误是不带缓冲的
* 若是指向终端设备的流，则是行缓冲的；否则是全缓冲的



如果不想使用上面默认的缓冲方式，可以使用下面的函数更改缓冲类型

```C
#include<stdio.h>

void setbuf(FILE *restrict fp, char *restrict buf);

int setvbuf(FILE *restrict fp, char *restrict buf, int mode, size_t size);
//成功返回 0； 出错返回 非0
```



这两个函数都应该在 **流已经打开 ** 且 **未进行任何其他操作**之前调用

怎么用？

`setbuf` 函数可以打开或关闭缓冲。`buf` 指向缓冲区，则打开缓冲；`buf` 为 `NULL` ，则关闭缓冲

`setvbuf` 可以使用 `mode` 参数说明缓冲类型

| `mode` | 缓冲类型 |
| ------ | -------- |
| _IOFBF | 全缓冲   |
| _IOLBF | 行缓冲   |
| _IONBF | 不带缓冲 |

  `size` 参数是用来指定缓冲区长度的

由于这两个函数的行为组合有点多，使用下面的表格进行说明

{% asset_img Snipaste_2021-08-03_09-19-29.png %}



怎么强制冲洗一个流？

使用 `fflush` 函数

```C
#include <stdio.h>

int fflush(FILE *fp);
// Returns: 0 if OK, EOF on error
```



## 5.5 打开流

有三个函数可以打开一个流

```C
#include <stdio.h>

FILE *fopen(const char *restrict pathname, const char *restrict type);

FILE *freopen(const char *restrict pathname, const char *restrict type,FILE *restrict fp);

FILE *fdopen(int fd, const char *type);
//Returns: file pointer if OK, NULL on error
```

三个函数区别如下：

* `fopen` 用于打开文件名指定的文件
* `freopen` 用于在一个**指定的流**上打开指定的文件
* `fdopen` 用于打开**文件描述符**指定的文件



`type` 参数指定打开流的方式（读/写/附加）

{% asset_img Snipaste_2021-08-03_09-28-41.png %}

调用 `fclose` 关闭一个打开的流

```C
#include <stdio.h>

int fclose(FILE *fp);
//Returns: 0 if OK, EOF on error
```



## 5.6 对一个流进行 读 和 写

1. **输入函数**

   一次读取**一个字符**的函数

   ```C
   #include <stdio.h>
   
   int getc(FILE *fp);
   
   int fgetc(FILE *fp);
   
   int getchar(void);
   //All three return: next character if OK, EOF on end of file or error
   ```

   `getchar` 相当于 `getc(stdin)` 。前两个函数的区别在于，`getc` 可以实现为宏，而 `fgetc` 是一个真正的函数

   可以看到，不论是出错还是到达文件末尾，那三个读函数都返回 `EOF` ，要想区分到底是哪种情况，还需调用下面这几个函数

   ```C
   #include <stdio.h>
   
   int ferror(FILE *fp);
   
   int feof(FILE *fp);
   //Both return: nonzero (true) if condition is true, 0 (false) otherwise
   
   void clearerr(FILE *fp);
   ```

   `clearerr` 用于清除 `FILE` 文件中维护的 **出错** 和 **文件结束** 这两个标志位

   

   如果读到了不满意的字符，想退货，把它再送回流中，可以使用下面这个函数

   ```C
   #include <stdio.h>
   
   int ungetc(int c, FILE *fp);
   //Returns: c if OK, EOF on error
   ```

   

2. **输出函数**

   ```C
   #include <stdio.h>
   
   int putc(int c, FILE *fp);
   
   int fputc(int c, FILE *fp);
   
   int putchar(int c);
   //All three return: c if OK, EOF on error
   ```

   它们的差别和上面的三个读取函数的差别是一样的



## 5.7 每次一行的 I/O

每次输入一行的函数

```C
#include <stdio.h>

char *fgets(char *restrict buf, int n, FILE *restrict fp);

char *gets(char *buf );
//Both return: buf if OK, NULL on end of file or error
```

`gets` 函数不推荐使用

`fgets` 总是将**换行符**写入缓冲区，而 `gets`  则是从不写入换行符



每次输出一行的函数

```C
#include <stdio.h>

int fputs(const char *restrict str, FILE *restrict fp);

int puts(const char *str);
//Both return: non-negative value if OK, EOF on error
```

总是推荐使用 `fgets`  和 `fputs` 

[使用 fgets 和 fputs 的例子](https://github.com/imzhangjinming/APUE/blob/master/5/fgets_fputs.c)



## 5.9 二进制 I/O

下面两个函数执行**二进制 I/O 操作**

```C
#include <stdio.h>

size_t fread(void *restrict ptr, size_t size, size_t nobj,FILE *restrict fp);

size_t fwrite(const void *restrict ptr, size_t size, size_t nobj,FILE *restrict fp);
//Both return: number of objects read or written
```



## 5.10 定位流

有三种方法定位标准 I/O 流

* `ftell` 和 `fseek` 函数。它们假定偏移量可以存储在一个**长整型**中
* `ftello` 和 `fseeko` 函数。使用 `off_t` 类型 代替了上面的长整型
* `fgetpos` 和 `fsetpos` 函数。

```C 
#include <stdio.h>

long ftell(FILE *fp);
//Returns: current file position indicator if OK, −1L on error

int fseek(FILE *fp, long offset, int whence);
//Returns: 0 if OK, −1 on error

void rewind(FILE *fp);
```

`ftell` 返回指定文件的偏移量， `fseek` 则用来设置偏移量

```C 
#include <stdio.h>

off_t ftello(FILE *fp);
//Returns: current file position indicator if OK, (off_t)−1 on error

int fseeko(FILE *fp, off_t offset, int whence);
//Returns: 0 if OK, −1 on error
```

这俩函数除了将 `long` 类型 换成 `off_t` 类型外，与 `ftell` 和 `fseek` 没有区别 



`fgetpos` 和 `fsetpos` 俩函数是 ISO C 标准引入的

```C
#include <stdio.h>

int fgetpos(FILE *restrict fp, fpos_t *restrict pos);

int fsetpos(FILE *fp, const fpos_t *pos);
//Both return: 0 if OK, nonzero on error
```



## 5.11 格式化 I/O

1. **格式化输出**

   ```C
   #include <stdio.h>
   
   int printf(const char *restrict format, ...);
   
   int fprintf(FILE *restrict fp, const char *restrict format, ...);
   
   int dprintf(int fd, const char *restrict format, ...);
   //All three return: number of characters output if OK, negative value if output error
   
   int sprintf(char *restrict buf, const char *restrict format, ...);
   //Returns: number of characters stored in array if OK, negative value if encoding error
   
   int snprintf(char *restrict buf, size_t n,const char *restrict format, ...);
   //Returns: number of characters that would have been stored in array
   //if buffer was large enough, negative value if encoding error
   ```

   `printf` 写入到标准输出， `fprintf` 写入到指定的流， `dprintf`  写入到文件描述符指定的文件；

   `sprintf`  写入到指定的 `buf` 中；`snprintf`  也是写入到指定的 `buf` 中  ，但是它指定了`buf` 的大小

    

    下面介绍如何编写转换说明。所谓转换说明，就是以上函数中的 `format` 函数，它决定了以什么样的格式显示后面的参数

   转换说明有四个可选部分

   ```shell
   %[flags][fldwidth][precision][lenmodifier]convtype
   ```

   `flags` 可选：

   {% asset_img Snipaste_2021-08-03_11-31-58.png %}

   `fldwidth` 说明最小字段宽度

   `precision` 以 `.` 开始，说明需要打印的精度

   `lenmodifier` 可选以下值

   {% asset_img Snipaste_2021-08-03_11-36-53.png %}

   

   `convtype` 不是可选的，而是**必选**的

   {% asset_img Snipaste_2021-08-03_11-38-47.png %}

   

2. **格式化输入**

   ```C
   #include <stdio.h>
   
   int scanf(const char *restrict format, ...);
   
   int fscanf(FILE *restrict fp, const char *restrict format, ...);
   
   int sscanf(const char *restrict buf, const char *restrict format, ...);
   //All three return: number of input items assigned,
   //EOF if input error or end of file before any conversion
   ```

   格式化输入同样需要转换说明

   ```shell
   %[*][fldwidth][m][lenmodifier]convtype
   ```

   这里只对 `convtype` 进行说明

   {% asset_img Snipaste_2021-08-03_11-44-59.png %}



## 5.12 实现细节

归根结底，标准I/O库还是要调用第三章所介绍的函数，但是第三章里的函数都是用 文件描述符 标识文件，而本章的函数大多使用 `FILE` 结构体，如何建立这两者之间的联系呢？答案是使用 `fileno` 函数

```C
#include <stdio.h>

int fileno(FILE *fp);
//Returns: the file descriptor associated with the stream
```



## 5.13 临时文件

```C
#include <stdio.h>

char *tmpnam(char *ptr);
//Returns: pointer to unique pathname

FILE *tmpfile(void);
//Returns: file pointer if OK, NULL on error
```

`tmpfile` 创建一个临时二进制文件，在关闭该文件或程序结束时将**自动删除**这种文件



[使用 tmpnam 和 tmpfile ](https://github.com/imzhangjinming/APUE/blob/master/5/tmpnam_tmpfile.c)



SUS 为处理临时文件定义了另外两个函数：

```C
#include <stdlib.h>

char *mkdtemp(char *template);
//Returns: pointer to directory name if OK, NULL on error

int mkstemp(char *template);
//Returns: file descriptor if OK, −1 on error
```



[使用 mkstemp ](https://github.com/imzhangjinming/APUE/blob/master/5/mkstemp_use.c)



## 5.14 内存流

内存流的特征：没有底层文件，所有的 I/O 操作都是通过缓存和主存之间的**字节运输**完成的

介绍第一个创建内存流的函数

```C
#include <stdio.h>

FILE *fmemopen(void *restrict buf, size_t size,const char *restrict type);
//Returns: stream pointer if OK, NULL on error
```

[使用 fmemopen 函数](https://github.com/imzhangjinming/APUE/blob/master/5/fmemopen_use.c)



# 课后习题

## 5.1 用 setvbuf 实现 setbuf

[我的解答](https://github.com/imzhangjinming/APUE/blob/master/5/exercise/mysetbuf.c)

