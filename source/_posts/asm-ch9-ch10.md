---
title: 汇编语言 第九、十章 含检测点和实验八、九、十
date: 2021-09-19 19:37:53
categories:
- notes
tags:
---

第九章 转移指令的原理

第十章 CALL 和 RET 指令



<!--more-->

## 第九章 转移指令的原理

转移指令就是能够控制CPU执行内存中某处代码的指令。包括可以修改 IP 或者同时修改 CS 和 IP 的指令

8086CPU 的转移行为有以下几类：

* 段**内**转移：只修改IP
  * **短**转移：IP的修改范围为 -128~127
  * **近**转移：IP的修改范围为 -32768~32767

* 段**间**转移：同时修改 CS 和 IP



8086CPU 的转移指令分为以下几类：

* 无条件转移指令
* 条件转移指令
* 循环指令
* 过程
* 中断



## 9.1 操作符 offset

这个操作符由**编译器**处理，功能是获得 **标号** 的 **偏移** 地址

```assembly
assume cs:codesg 

codesg segment 
	start:	mov ax,offset start		;相当于mov ax，0
	s:		mov ax,offset s			;相当于mov ax，3
codesg ends 

end start
```



## 9.2 jmp 指令

jmp 可以只修改 IP，也可以同时修改 CS 和 IP

jmp 指令需要给出**两种**信息：

* 转移的**目的**地址
* 转移的距离



## 9.3 根据位移进行转移的 jmp 指令

`jmp short 标号`

含义：转移到标号处执行指令

这种格式是 **段内短转移**，IP 修改范围是 -128~127 ，也就是说，最多往前跳128字节，往后跳127字节



`jmp near ptr 标号`

（1）16位位移 = 标号处的地址 - jmp指令后的**第一个字节**的地址；
（2）near ptr 指明此处的位移为16位位移，进行的是**段内近转移**；
（3）16位位移的范围为 **-32768~32767**，用**补码**表示；
（4）16位位移由编译程序在编译时算出。



## 9.4 转移的目的地址在指令中的 jmp 指令

`jmp far ptr 标号`

实现**段间转移**



## 9.5 转移地址在寄存器中的 jmp 指令

`jmp 16位寄存器`

功能：(IP) = (16位寄存器) 



## 9.6 转移地址在内存中的 jmp 指令

* `jmp word ptr 内存单元地址`

**段内转移**，指定的那个内存单元里的内容就是目的地址

```ASS
mov ax,0123H 
mov ds:[0],ax 
jmp word ptr ds:[0]
```

执行后，(IP) = 0123H



* `jmp dword ptr 内存单元地址`

**段间转移**

(CS)=(内存单元地址+2)

(IP)=(内存单元地址)

```assembly
mov ax,0123H 
mov ds:[0],ax 
mov word ptr ds:[2],0
jmp dword ptr ds:[0]
```

执行后，(CS)=0000H，(IP)=0123H



### 检测点 9.1 

1. 程序如下

```assembly
assume cs:code 

data segment
	db 0,offset start			;answer
data ends 

code segment 
	start:	mov ax,data 
			mov ds,ax 
			mov bx,0
			jmp word ptr [bx+1]
code ends

end start
```

若要使程序中的 jmp 指令执行后，CS:IP 指向程序的第一条指令，在 data 段中应该定义哪些数据？ 



2. 程序如下

```assembly
assume cs:code 

data segment
	dd 12345678H 
data ends 

code segment 
	start:	mov ax,data 
			mov ds,ax 
			mov bx,0
			mov word ptr [bx],offset start		;answer 
			mov [bx+2],cs 						;answer
			jmp dword ptr ds:[0]
code ends

end start
```

补全程序，使 jmp 指令执行后，CS:IP 指向程序的第一条指令



3. 用 debug 查看内存，结果如下

2000:1000 BE 00 06 00 00 00 ···

则此时，CPU执行指令：

```assembly
mov ax,2000H 
mov es,ax 
jmp dword ptr es:[1000H]
```

后，(CS)=0006H , (IP)=00BEH



## 9.7 jcxz 指令

jcxz 指令为**有条件转移**指令，所有的有条件转移指令都是**短转移**，在对应的机器码中包含转移的位移，而不是目的地址。对 IP 的修改范围都为：**-128~127**

`jcxz 标号`

如果 `(cx)=0` ，跳转到标号处执行

如果  `(cx)!=0` ，什么也不做



### 检测点 9.2

补全编程，利用 jcxz 指令，实现在内存 2000H 段中查找第一个值为 0 的字节，找到后，将它的偏移地址存储在dx中

```assembly
assume cs:code 

code segment 
	start:	mov ax,2000H 
			mov ds,ax 
			mov bx,0
	s:		mov ch,0		;answer
			mov cl,[bx]		;answer
			jcxz ok			;answer
			inc bx			;answer
			jmp short s
			
	ok:		mov dx,bx
			
			mov ax,4c00h 
			int 21h 
code ends

end start		
```



## 9.8 loop 指令

loop 指令为循环指令，所有的循环指令都是**短转移**



### 检测点 9.3 

补全编程，利用 loop 指令，实现在内存 2000H 段中查找第一个值为 0 的字节，找到后，将它的偏移地址存储在  

 dx 中。

```assembly
assume cs:code 

code segment 
	start:	mov ax,2000H 
			mov ds,ax 
			mov bx,0
	s:		mov cl,[bx]
			mov ch,0
			inc cx		;answer
			inc bx 
			loop s 
	ok:		dec bx		;dec指令的功能和inc相反，dec bx进行的操作为：（bx）=（bx）-1
			mov dx,bx 
			
			mov ax,4c00h
			int 21h 
code ends

end start		
```





## 9.9 根据位移进行转移的意义

这种设计，方便了程序段在内存中的浮动装配

因为给出的跳转位移是**相对的**



## 9.10 编译器对转移位移超界的检测

> 注意，根据位移进行转移的指令，它们的转移范围受到转移位移的限制，如果在源程序中出现了转移范围超界的问题，在编译的时候，编译器将报错。



# 实验八 分析一个奇怪的程序

```assembly
assume cs:codesg 

codesg segment 
		mov ax,4c00h 
		int 21h 
start:	mov ax,0
s:		nop 
		nop 
		
		mov di,offset s 
		mov si,offset s2
		mov ax,cs:[si]
		mov cs:[di],ax 

s0:		jmp short s 

s1:		mov ax,0
		int 21h 
		mov ax,0

s2:		jmp short s1
		nop 

codesg ends 

end start
```

运行前思考，这个程序可以正确返回吗？

不能

上机运行看看：

和我想的不一样，下面分析一下

{% asset_img Snipaste_2021-09-18_08-52-40.png %}

注意 076A:0020 处 JMP 指令的机器码 **EBF6** 

F6 是 -10 的补码，为什么是 -10 呢，我们看这段程序

```assembly
s1:		mov ax,0
		int 21h 
		mov ax,0

s2:		jmp short s1	;EBF6 对应的汇编指令
		nop 
```

从内存分布图中可以看到， `mov ax,0` `int 21h` `mov ax,0` 各占用了4、2、4字节内存，所以从 `jmp short s1` 处跳转到 s1 处要**往前**转移 **10 字节**，所以机器码中是 F6 (-10)

继续运行程序

运行完 `		mov cs:[di],ax ` 这一行后，内存分配如下图所示，重点来了

{% asset_img Snipaste_2021-09-18_08-59-40.png %}

原来的两个 nop 指令被替换为了  **EBF6** ，好家伙，原来是**直接拷贝机器码**

找到问题的根源了，运行时候的代码复制只能复制编译后的机器码，所以这里复制完后，标号 s 处的机器指令含义是 往前跳转 10个 字节，正好能够跳转到 `mov ax,4c00h ` 处，程序正常退出



# 实验九 根据材料编程

编程：在屏幕中间分别显示**绿色**、**绿底红色**、**白底蓝色**的字符串 ‘welcome to masm!'

**资料**：

80×25彩色字符模式显示缓冲区（以下简称为显示缓冲区）的结构：

内存地址空间中，**B8000H~BFFFFH** 共**32KB**的空间，为80×25彩色字符模式的显示缓冲区。向这个地址空间写入数据，写入的内容将**立即**出现在显示器上。

在80×25彩色字符模式下，显示器可以显示**25行**，**每行80个**字符，每个字符可以有**256种属性**（背景色、前景色、闪烁、高亮等组合信息）。

这样，一个字符在显示缓冲区中就要占**两个字节**，分别存放字符的ASCII码和属性。80×25模式下，**一屏**的内容在显示缓冲区中共占**4000个字节**。

显示缓冲区分为8页，每页4KB（~4000B），显示器可以显示任意一页的内容。一般情况下，显示**第0页**的内容。也就是说通常情况下，**B8000H~B8F9FH** 中的4000个字节的内容将出现在显示器上。



在一页显示缓冲区中：

偏移**000~09F**对应显示器上的**第1行**（80个字符占160个字节）；

偏移0A0~13F对应显示器上的第2行；

偏移140~1DF对应显示器上的第3行；

依此类推，可知，偏移F0O~F9F对应显示器上的第25行。



在一行中，一个字符占两个字节的存储空间（一个字），**低位**字节存储字符的**ASCII码**，**高位**字节存储字符的**属性**。一行共有80个字符，占160个字节。

即在一行中：
**00~01单元**对应显示器上的**第1列**；

02~03单元对应显示器上的第2列；

04~05单元对应显示器上的第3列；



一个在屏幕上显示的字符，具有前景（字符色）和背景（底色）两种颜色，字符还可以以高亮度和闪烁的方式显示。前景色、背景色、闪烁、高亮等信息被记录在属性字节中。



属性字节的格式：

{% asset_img Snipaste_2021-09-18_09-27-44.png %}

可以按位设置属性字节，从而配出各种不同的前景色和背景色

比如：

红底绿字，属性字节为：01000010B；

红底闪烁绿字，属性字节为：11000010B；

红底高亮绿字，属性字节为：01001010B；

黑底白字，属性字节为：00000111B；

白底蓝字，属性字节为：01110001B。



**资料到此结束**



先来确定每行字符的属性字节：

第一行黑底绿色：0000 0010B （02H）

第二行绿底红字：0010 0100B （24H）

第三行白底蓝字：0111 0001B （71H）

welcome to masm! 一共16个字符，需要占用32个字节。它们的 ASCII 码分别是

77 65 6c 63 6f 6d 65 20 74 6f  20 6d 61 73 6d 21

所以，第一行需要写入的数据是

77 02 65 02 6c 02 63 02 6f 02 6d 02 65 02 20 02 74 02 6f  02 20 02 6d 02 61 02 73 02 6d 02 21 02

第二行

77 24 65 24 6c 24 63 24 6f 24 6d 24 65 24 20 24 74 24 6f  24 20 24 6d 24 61 24 73 24 6d 24 21 24

第三行

77 71 65 71 6c 71 63 71 6f 71 6d 71 65 71 20 71 74 71 6f  71 20 71 6d 71 61 71 73 71 6d 71 21 71



题目要求在屏幕中间显示这三行，所以应该写入第 12~14 行 的 第 33~48 列，计算每行**写入**的起始地址

06E0:40 

0780:40

0820:40

用 es 寄存器存放段地址，(es)=B800，那么寻址方式如下

es:06e0H[si]

es:0780H[si]

es:0820H[si]

(si) 从 40H 开始，每次增加 2，循环 16 次就能写入一行 

下面开始编程

```assembly
assume cs:codesg,ds:data

data segment
	dw 7702h, 6502h, 6c02h, 6302h, 6f02h, 6d02h, 6502h, 2002h, 7402h, 6f02h, 2002h, 6d02h, 6102h, 7302h, 6d02h, 2102h
	dw 7724h, 6524h, 6c24h, 6324h, 6f24h, 6d24h, 6524h, 2024h, 7424h, 6f24h, 2024h, 6d24h, 6124h, 7324h, 6d24h, 2124h
	dw 7771h, 6571h, 6c71h, 6371h, 6f71h, 6d71h, 6571h, 2071h, 7471h, 6f71h, 2071h, 6d71h, 6171h, 7371h, 6d71h, 2171h
data ends

codesg segment
	start:	mov ax,data
			mov ds,ax
			mov bx,0
			
			mov ax,0B800H
			mov es,ax
			
			mov si,40h
			
			mov cx,16
		s:	mov al,ds:[bx]
			mov ah,ds:[bx+1]
			mov es:06e0H[si],ah
			mov es:06e0H[si+1],al

			mov al,ds:32[bx]
			mov ah,ds:32[bx+1]
			mov es:0780H[si],ah
			mov es:0780H[si+1],al
			
			mov al,ds:64[bx]
			mov ah,ds:64[bx+1]
			mov es:0820H[si],ah
			mov es:0820H[si+1],al
			
			add bx,2
			add si,2
			
			loop s
			
			mov ax,4c00h
			int 21h
			
codesg ends

end start
```

运行结果：

{% asset_img Snipaste_2021-09-18_11-12-38.png %}





# 第十章 CALL 和 RET 指令

call 和 ret 指令都是**转移指令**，它们都**修改 IP**，或**同时修改 CS 和 IP **。它们经常被共同用来实现**子程序的设计**。



## 10.1 ret 和 retf

**ret** 指令用栈中的数据，修改IP的内容，从而实现**近转移**；

**retf** 指令用栈中的数据，修改CS和IP的内容，从而实现**远转移**。

CPU执行 ret 指令时，相当于进行：

```assembly
 POP IP
```

CPU执行 retf 指令时，相当于进行：

```assembly
POP IP
pop CS
```



### 检测点 10.1 

补全程序，实现从内存 1000:0000 处开始执行指令

```assembly
assume cs:code 

stack segment
	db 16 dup (0)
stack ends 

code segment 
	start:	mov ax,stack 
			moV ss,ax
			mov sp,16
			mov ax,1000H	;ANSWER
			push ax
			mov ax,0		;ANSWER
			push ax 
			retf 
code ends

end start
```



## 10.2 call 指令

CPU执行call指令时，进行两步操作：

（1）将当前的IP或CS和IP压入栈中；
（2）转移。

call 指令**不能**实现**短转移**，除此之外，call 指令 实现转移的方法和 jmp 指令的原理相同



## 10.3 依据位移进行转移的 call 指令

`call 标号`

将当前的 IP 压入栈中，转到标号处执行

如果我们用汇编语法来解释此种格式的call指令，则：

CPU执行 `call 标号` 时，相当于进行：

```assembly
push IP
jmp near ptr 标号
```



### 检测点 10.2

下面的程序执行后，ax 中的数值是多少？

| 内存地址 | 机器码 | 汇编指令 |
| :--------: | :------: | :--------: |
| 1000:0 | B8 00 00 | mov ax,0 |
| 1000:3 | E8 01 00 | call s |
| 1000:6 | 40 | inc ax |
| 1000:7 | 58 | s:pop ax |

(ax)=6



## 10.4 转移的目的地址在指令中的 call 指令

`call far ptr 标号`

实现**段间转移**

如果我们用汇编语法来解释此种格式的 call 指令，则：

CPU执行 `call far ptr 标号` 时，相当于进行：

```assembly
push CS
push IP
jmp far ptr 标号
```





### 检测点 10.3

下面的程序执行后，ax 中的数值是多少？

| 内存地址 | 机器码 | 汇编指令 |
| :--------: | :------: | :--------: |
| 1000:0 | B8 00 00 | mov ax,0 |
| 1000:3 | 9A 09 00 00 10 | call far ptr s |
| 1000:8 | 40 | inc ax |
| 1000:9 | 58 | s:pop ax |
|  |  | add ax,ax |
|  |  | pop bx |
|  |  | add ax,bx |

(ax)=1010h



## 10.5 转移地址在寄存器中的 call 指令

`call 16位reg`

用汇编语法来解释此种格式的 call 指令，CPU 执行 `call 16位reg`  时，相当于进行：

```assembly
push IP
jmp 16位reg
```



### 检测点 10.4 

下面的程序执行后，ax 中的数值是多少？

| 内存地址 | 机器码 | 汇编指令 |
| :--------: | :------: | :--------: |
| 1000:0 | B8 06 00 | mov ax,6 |
| 1000:3 | FF D0 | call ax |
| 1000:5 | 40 | inc ax |
| 1000:6 | | mov bp,sp |
|  | | add ax,[bp] |

(ax)=Bh



## 10.6 转移地址在内存中的 call 指令

* `call word ptr 内存单元地址`

相当于

```assembly
push IP
jmp word ptr 内存单元地址
```



* `call dword ptr 内存单元地址`

相当于

```assembly
push CS
push IP
jmp dword ptr 内存单元地址
```



### 检测点 10.5

1. 下面的程序执行后， ax 中的数值是多少？

```assembly
assume cs:code 

stack segment 
	dw 8 dup (0)
stack ends 

code segment 
	start:	mov ax,stack 
            mov ss,ax
            mov sp,16
            mov ds,ax 
            mov ax,0
            call word ptr ds:[0EH]
            inc ax
            inc ax
            inc ax 

            mov ax,4c00h 
            int 21h 
code ends

end start
```

(ax)=3

2. 下面的程序执行后， ax 和 bx 中的数值是多少？

```assembly
assume cs:code 

data segment
	dw 8 dup (0)
data ends 

code segment 
	start:	mov ax,data 
			mov ss,ax
			mov sp,16
			mov word ptr ss:[0],offset s 
			mov ss:[2],cs 
			call dword ptr ss:[0]
			nop 
		s:	mov ax,offset s 
			sub ax,ss:[0cH]			;(ax)=1
			mov bx,cs 
			sub bx,ss:[0eH]			;(bx)=0
			
			mov ax,4c00h 
			int 21h 
code ends

end start
```

(ax)=1 ; (bx)=0



## 10.7 call 和 ret 的配合使用

这两个指令配合使用可以编写**子程序**



下面的程序返回前，bx 中的值是多少？

```assembly
assume cs:code 

code segment 
	start:	mov ax,1
			mov cx,3
			call s 
			mov bx,ax			;(bx)=8
			mov ax,4c00h 
			int 21h 
	s:		add ax,ax
			1oops 
			ret 
code ends

end start
```



## 10.8 mul 指令

乘法指令

（1）两个相乘的数：两个相乘的数，要么**都是8位**，要么**都是16位**。如果是8位，一个默认放在AL中，另一个放在8位 reg 或 内存字节单元 中；如果是16位，一个默认在AX中，另一个放在 16 位 reg 或 内存**字**单元 中。

（2）结果：如果是8位乘法，结果默认放在AX中；如果是16位乘法，结果**高位**默认在**DX**中存放，**低位**在**AX**中放。



## 10.9 模块化程序设计



## 10.10 参数和结果的传递问题

比如，设计一个子程序，可以根据提供的N，来计算N的3次方。

这里面就有两个问题：

（1）将**参数N存储**在什么地方？

（2）计算得到的**结果**，**存储**在什么地方？

可以用**寄存器**来传递

```assembly
assume cs:code 
data segment 
	dw	1,2,3,4,5,6,7,8
	dd 	0,0,0,0,0,0,0,0
data ends

code segment 
	start:	mov ax,data 
			mov ds,ax 
			mov si,0		;ds:si指向第一组word单元
			mov di,16		;ds:di指向第二组dword单元
			mov cx,8
	s:		mov bx,[si]
			call cube 
			mov [di],ax 
			mov [di].2,dx 
			add si,2		;ds:si指向下一个word单元
			add di,4		;ds:di指向下一个dword单元
			1oop s 
			
			mov ax,4c00h 
			int 21h 
	cube:	mov ax,bx 
			mul bx 
			mul bx 
			ret 
			
code ends

end start
```



## 10.11 批量数据的传递

当需要传递的数据很多时，用有限的寄存器来传递就显得不合适了

但是内存是很多的，比寄存器多得多，所以可以考虑把数据存到一块内存中，然后把这块内存的**首地址放在寄存器**中



编程，将 data 段中的字符串转化为大写

```assembly
assume cs:code 

data segment 
	db 'conversation'
data ends 

code segment 
	start:	mov ax,data 
			mov ds,ax 
			mov si,0		;ds:si指向字符串（批量数据）所在空间的首地址
			mov cx,12		;cx存放字符串的长度
			call capital
			
			mov ax,4c00h 
			int 21h 
	capital:and byte ptr[si],11011111b 
			inc si 
			loop capital 
code ends

end start
```



## 10.12 寄存器冲突的问题

所谓寄存器冲突问题，就是主程序和子程序都要使用某些寄存器从而相互干扰，使程序无法正常运行

解决这个问题的通用方法是在子程序开头，把子程序要用到的所有寄存器内容入栈，待子程序完成后，再把之前入栈的寄存器内容出栈，然后返回



# 实验10 编写子程序

### 一、显示字符串

问题：显示字符串是现实工作中经常要用到的功能，应该编写一个通用的子程序来实现这个功能。我们应该提供灵活的调用接口，使调用者可以决定显示的**位置**（行、列）、**内容**和**颜色**。



子程序描述：

​	名称：show_str

​	功能：在指定的位置，用指定的颜色，显示一个**用 0 结束**的字符串

​	参数：(dh)=行号（取值范围0~24），(dl)=列号（取值范围0~79），(cl)=颜色（取值范围0~7），ds:si 指向字符串的首地址

​	返回：无

​	应用举例：在屏幕的8行3列，用绿色显示data段中的字符串

```assembly
assume cs:code 

data segment
	db 'Welcome to masm!',0
data ends 

stack segment
	dw 30 dup (0)
stack ends

code segment 
	start: 		mov dh,8
                mov dl,3
                mov cl,2
                mov ax,data 
                mov ds,ax 
                mov si,0
                
                mov ax,stack
                mov ss,ax
                mov sp,3ch
                
                call show_str 

                mov ax,4c00h 
                int 21h 
	show_str:	push cx
				push ax
				;push si
				push es
				;push ds
				push dx
				push di
				push bp
				
				;mov ax,data
				;mov ds,ax
				;mov bx,0
				
				mov ax,0b800h
				mov es,ax

                mov al,160
				mul dh
				mov bp,ax
				
				mov al,2
				mul dl
				mov di,ax
				
	wr_ch:		push cx
				mov cl,ds:[si]
				mov ch,0
				jcxz ok
				mov es:[bp][di],cl				
				pop cx
                mov es:[bp][di+1],cl

				add si,1
				add di,2
				jmp short wr_ch
				
	ok:			pop cx
                pop bp
				pop di
				pop dx
				;pop ds
				pop es
				;pop si
				pop ax
				pop cx
				ret
	
code ends

end start
```

运行效果：

{% asset_img Snipaste_2021-09-19_11-02-56.png %}

修改参数，在第20行，75列，显示蓝色 'Hello masm , glad to see you!'

修改部分的代码如下：

{% asset_img Snipaste_2021-09-19_11-04-52.png %}

运行效果：

{% asset_img Snipaste_2021-09-19_11-05-56.png %}

蓝色不太清晰，换成红色试试：

{% asset_img Snipaste_2021-09-19_11-06-52.png %}



### 二、解决除法溢出问题

8.7 节讲过的 div 指令

本应该存储运算结果的寄存器存放不下结果，就会发生除法溢出问题



现在我们要编写一个子程序来进行不会产生溢出的除法运算



**程序描述**

​	名称：divdw

​	功能：进行不会产生溢出的除法运算，被除数为 dword 型，除数为 word 型，结果为 dword 型


​	参数：（ax）=dword型数据的低16位；（dx）=dword型数据的高16位；（cx）=除数

​	返回：（ax）=结果的低16位；（dx）=结果的高16位；（cx）=余数

​	应用举例：计算 1000000/10（F4240H/0AH)

```assembly
mov ax,4240H
mov dx,000FH 
mov cx,0AH
call divdw
```

​	结果：(dx)=0001H ,(ax)=86A0H ,(cx)=0

```assembly
assume cs:codesg

codesg segment
	start:	mov ax,4240H
			mov dx,000FH 
            mov cx,0AH
            call divdw
            
            mov ax,4c00h
            int 21h
	divdw:	push bx
			push ax
			
			mov ax,dx
			mov dx,0
			div cx			;现在高16位做除法的商在AX中，余数在DX中
			mov bx,ax		;保存高16位除法的商
			
			pop ax
			div cx			;现在低16位做除法的商在AX中
			
			mov dx,bx
			mov cx,dx
			
			pop bx
			ret
			
codesg ends

end start			
```



### 三、数值显示

**问题**

编程，将 data 段中的数据以十进制的形式显示出来

```assembly
data segment 
	dw 123,12666,1,8,3,38
data ends
```

要想完成这个任务，需要进行两步工作：

1. 将用二进制信息存储的数据转变为十进制形式的字符串
2. 显示十进制形式的字符串

第二步调用本次实验的第一个程序 show_str 就OK了

重要的是如何完成第一步

**程序描述**

​	名称：dtoc

​	功能：将 word 型数据转变为表示十进制数的字符串，字符串以 0 为结尾符

​	参数：(ax) = word 型数据；ds:si  指向字符串的首地址

​	返回：无

​	应用举例：编程，将数据12666以十进制的形式在屏幕的 8 行 3 列，用绿色显示出来

```assembly
assume cs:code

data segment
	db 10 dup (0)
data ends

stack segment
    db 100 dup (0)
stack ends

code segment 
	start:	mov ax,12666
			mov bx,data 
			mov ds,bx 
			mov si,0

            mov bx,stack
            mov ss,bx
            mov sp,100

			call dtoc 
			
			mov dh,8
			mov dl,3
			mov cl,2
			call show_str

            mov ax,4c00h
            int 21h
            
	dtoc:	push dx
			push cx
	itra:	mov dx,0
			mov cx,10
            div cx			;商在 AX 中，余数在 DX 中
			add dx,30h
			push dx
			inc si
			
			mov cx,ax
			jcxz dtoc_ok

			jmp short itra
	dtoc_ok:mov byte ptr ds:[si],0	
    		mov cx,si
			mov si,0
	s:		pop dx
			mov ds:[si],dl
			inc si
			loop s
			mov si,0
			
			pop cx
			pop dx
			ret
	
	show_str:	push cx
				push ax
				;push si
				push es
				;push ds
				push dx
				push di
				push bp
				
				;mov ax,data
				;mov ds,ax
				;mov bx,0
				
				mov ax,0b800h
				mov es,ax

                mov al,160
				mul dh
				mov bp,ax
				
				mov al,2
				mul dl
				mov di,ax
				
	wr_ch:		push cx
				mov cl,ds:[si]
				mov ch,0
				jcxz ok
				mov es:[bp][di],cl				
				pop cx
                mov es:[bp][di+1],cl

				add si,1
				add di,2
				jmp short wr_ch
				
	ok:			pop cx
                pop bp
				pop di
				pop dx
				;pop ds
				pop es
				;pop si
				pop ax
				pop cx
				ret
    		
code ends

end start
```

运行结果：

{% asset_img Snipaste_2021-09-19_14-30-47.png %}

