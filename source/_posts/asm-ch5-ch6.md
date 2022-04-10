---
title: 汇编语言 第五章 第六章 含检测点和实验四、五
date: 2021-09-11 18:06:12
categories:
- notes
tags:
---
第五章 [BX] 和 loop 指令

第六章 包含多个段的程序



<!--more-->



# 第五章 [BX] 和 loop 指令

## 5.1 [BX]

`mov ax,[bx]`

功能：bx 中存放的数据作为一个**偏移地址**EA，段地址SA默认在ds中，将 SA:EA 处的数据送入ax中。即：(ax)=((ds)*16+(bx))

`mov [bx],ax`

功能：bx 中存放的数据作为一个 **偏移地址 **EA，段地址SA默认在ds中，将ax中的数据送入内存 SA:EA 处。即：((ds)*16+(bx))=(ax)



## 5.2 Loop 指令

loop 指令实现循环的程序框架基本如下：

```assembly
	mov cx,循环次数
s:
	循环执行的程序段
	loop s
```

cx 中存放的是循环次数，s 是希望循环的程序段的标号，loop s 执行循环



**问题5.2**

编程，用加法计算123*236，结果存在ax中。

```assembly
codesg segment
	mov ax,0
	mov cx,236
s:
	add ax,123
	loop s
	
	mov ax,4c00h
	int 21h
	
codesg ends
end
```



**问题 5.3**

改进程序5.2，提高123*236的计算速度。

```assembly
codesg segment
	mov ax,123
	mov cx,8
s:
	add ax,ax
	loop s
	mov cx,20
r:
	sub ax,123
	loop r
	
	mov ax,4c00h
	int 21h
	
codesg ends
end
```



## 5.3 在 debug 中跟踪用 Loop 指令实现的循环程序

> 我们知道，大于9FFFh的十六进制数据A000H、A001H...C000H、C001H...FFFEH、FFFFH等，在书写的时候都是以字母开头的。而在汇编源程序中，数据**不能以字母开头**，所以要在前面加0。比如，9138h在汇编源程序中可以直接写为“9138h”，而A000h在汇编源程序中要写为“0A000h”。

介绍一个新的 debug 命令—— g

`g 偏移地址` 可以将程序执行到 **CS:偏移地址** 处



使用 `p` 命令可以**跳出循环**



## 5.4 debug 和 masm 对指令的不同处理

debug 中 `mov ax,[0]` 被解释为 (ax)= DS:0 地址处的内容

masm 中  `mov ax,[0]` 被解释为 (ax)=0

如果要在masm 中指定内存单元，需要在 [0] 前面加上**段寄存器名字**，比如 `mov ax,ds:[0]` ; 

或者先将偏移地址送入一个寄存器，比如 bx ，然后再   `mov ax,[bx]` 



## 5.5 loop 和 [bx] 的联合应用

考虑这样一个问题，计算ffff：0-ffff:b单元中的数据的和，结果存储在dx中。

```assembly
codesg segment
	mov ax,0ffffh
	mov ds,ax
	mov bx,0
	mov dx,0
	
	mov cx,12
s:
	mov al,ds:[bx]
	mov ah,0
	add dx,ax
	add bx,1
	
	loop s
	
	mov ax,4c00h
	int 21h
	
codesg ends
end
```



## 5.6 段前缀

5.4节介绍过的  `mov ax,ds:[0]`  指令中，用于**显式地**指明内存单元的段地址的 **ds** 称为 **段前缀**



## 5.7 一段安全的空间

> （3）DOS方式下，一般情况，0:200~0:2ff空间中没有系统或其他程序的数据或代码；

也就是说，一般可以向 0:200~0:2ff 这段空间写入

验证过，这段内存的内容确实都是  0

{% asset_img Snipaste_2021-09-10_09-00-22.png %}

## 5.8 段前缀的使用



# 实验四 [bx] 和 loop 的使用

1. 编程，向内存 0:200~0:23F 依次传送数据 0~63 (3FH)

2. 编程，向内存 0:200~0:23F 依次传送数据 0~63 (3FH)，程序中只能使用9条指令，9条指令中包括 `mov ax，4c00h` 和 `int 21h`

   1和2一起解答：

   ```assembly
   assume cs:codesg
   
   codesg segment
       mov ax,0020h
       mov ds,ax
       mov bx,0
   
       mov cx,64
   
   s:  mov ds:[bx],bl
       inc bx
       loop s
   
       mov ax,4c00h
       int 21h
       
   codesg ends
   end
   ```

   运行结果：

   {% asset_img Snipaste_2021-09-10_09-25-46.png %}

3. 下面的程序的功能是将 `mov ax,4c00h` 之前的指令复制到内存 0:200 处，补全程序。上机调试，跟踪运行结果

```assembly
assume cs:code
code segment
	mov ax,cs
	mov ds,ax
	mov ax,0020h
	mov es,ax
	mov bx,0
	mov cx,23
s:	mov al,[bx]
	mov es:[bx],al
	inc bx
	loop s
	mov ax,4c00h
	int 21h
code ends
end
```



# 第六章 包含多个段的程序

**为什么**一个程序要包含**多个段**？

从**内存空间的获取**角度来讲，我们通过**定义段**的方式在程序**被加载**的时候**获取**内存空间。（当然也可以在程序运行的时候获得内存空间，但是书中不会涉及这方面的知识）

从**程序规划**的角度来看，大多数有用的程序都要处理**数据**、使用**栈空间**、使用**指令**，为了程序的清晰和方便，一般会定义**不同的段**来存放它们。

## 6.1 在代码段中使用数据

要在代码段中使用数据，首先要在代码段中**定义数据**，就像下面这样：

```assembly
assume cs:code	
code segment 
	dw 0123h,0456h,0789h,0abch,0defh,0fedh,0cbah,0987h
	mov bx,0
	mov ax,0
	mov cx,8
s:	add ax,cs:[bx]
	add bx,2
	1oop s
	
	mov ax,4c00h
	int 21h
code ends

end
```

`dw 0123h,0456h,0789h,0abch,0defh,0fedh,0cbah,0987h` 这一行代码就是定义数据的，`dw` 是 define word 定义字的意思

这样编译出来的程序有一个**问题**，就是定义的那些数据位于**代码段的开始**，它们不是指令，所以**没法被执行**，这样运行肯定会出错。除非我们用 debug 指定开始执行的地址（即修改 CS:IP），但是这不是解决问题的根本办法，因为总有程序，或者说绝大多数程序是直接运行的（不用 debug.exe）



那怎么办呢？

我们可以在源程序中指定程序开始的地方，像下面这样：

```assembly
assume cs:code
code segment
	dw 0123h,0456h,0789h,0abch,Odefh,0fedh,Ocbah,0987h
	start:	mov bx,0
			mov ax,0
			mov cx,8
	s:		add ax,cs:[bx]
			add bx,2
			1oop s
			
			mov ax,4c00h
			int 21h
code ends
end start
```

注意第4行和第14行，`start:` 和 `end start` 就定义了程序的入口。这里 `start` **只是一个标号**，可以把它换成你喜欢的单词，只要记得在  `end` 后面加上**对应的标号**就可以了。比如  `war:` 和 `end war`  也能定义程序的入口（而且在程序执行后可以“结束战争”，让地球重回和平）



## 6.2 在代码段中使用栈

如何利用栈将下面程序里定义的数据逆序存放？

```assembly
assume cs:codesg

codesg segment
	dw 0123h,0456h,0789h,0abch,0defh,0fedh,0cbah,0987h
	?
codesg ends

end
```

可以用一个栈，先将这些数据依次入栈，然后在依次出栈。当然从入栈转换到出栈之前需要调整一下接受出栈数据的地址

如何在程序中申请一个栈？

可以定义一系列空白数据，所有数据的大小就是你想要的栈的大小。比如这样：

`dw 0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0`

程序被加载到内存之后我们就能把这段地址当作栈来用了

所以，解决本节开头的那个问题的程序可以这样写：

```assembly
assume cs:codesg
codesg segment
	dw 0123h,0456h,0789h,0abch,0defh,0fedh,0cbah,0987h
	dw 0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0
	;用dw定义16个字型数据，在程序加载后，将取得16个字的；内存空间，存放这16个数据。在后面的程序中将这段；空间当作栈来使用
	
	start:	mov ax,cs
			mov ss,ax
			mov sp,30h	;将设置栈顶ss:sp指向cs：30
			mov bx,0
			mov cx,8
	s:		push cs:[bx]
			add bx,2
			1oops		;以上将代码段0~15单元中的8个字型数据依次入栈
			mov bx,0
			mov cx,8
	s0:		pop cs:[bx]
			add bx,2
			1oop s0		;以上依次出栈8个字型数据到代码段0~15单元中
			
			mov ax,4c00h
			int 21h
codesg ends
end start				;指明程序的入口在start处		
```



## 检测点 6.1

1. 下面的程序实现依次用内存 0:0~0:15 单元中的内容改写程序中的数据，完成程序：

```assembly
assume cs:codesg
codesg segment
	dw 0123h,0456h,0789h,0abch,0defh,0fedh,0cbah,0987h
	start:	mov ax,0
			mov ds,ax
			mov bx,0
			mov cx,8
	s:		mov ax,[bx]
			
			mov cs:[bx],ax		; answer
			
			add bx,2
			1oop s
			
			mov ax,4c00h
			int 21h
codesg ends
end start
```

2. 下面的程序实现依次用内存 0:0~0:15 单元中的内容改写程序中的数据，数据的传送用**栈**来进行。栈空间设置在程序内。完成程序：

```assembly
assume cs:codesg
codesg segment
		dw 0123h,0456h,0789h,0abch,0defh,0fedh,0cbah,0987h
		dw 0,0,0,0,0,0,0,0,0,0	;10个字单元用作栈空间
		
		start:	mov ax,cs		; answer
				mov ss,ax
				mov sp,24h		; answer
				
				mov ax,0
				mov ds,ax 
				mov bx,0
				mov cx,8
		s:		push [bx]
				
				pop cs:[bx]		; answer
				
				add bx,2
				loop s
				
				mov ax,4c00h
				int 21h
codesg ends
end start
```



## 6.3 将数据、代码、栈放入不同的段

这一章前两节的代码中都把数据、栈和指令放在一个段里了，这样的坏处很明显：计算地址很麻烦

需要自己来算数据从何处开始，何处结束，对于栈也是如此

我们为什么不给每一个部分起一个名字，直接用它们的名字进行寻址呢？

这种做法就是 把 数据、代码、栈 放入不同的段，看下面这段代码：

```assembly
assume cs:code,ds:data,ss:stack

data segment
	dw 0123h,0456h,0789h,0abch,0defh,0fedh,0cbah,0987h 
data ends

stack segment 
	dw 0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0
stack ends

code segment 
	start:	mov ax,stack
			mov ss,ax
			mov sp,20h		;设置栈顶ss:sp指向stack:20
			mov ax,data 
			mov ds,ax		;ds指向data段
			mov bx,0		;ds:bx指向data段中的第一个单元
			mov cx,8
	s:		push [bx]
			add bx,2
			loop s			;以上将data段中的0~15单元中的8个字型数据依次入栈
			
			mov bx,0
			mov cx,8
	s0:		pop [bx]
			add bx,2
			loop s0			;以上依次出栈8个字型数据到data段的0~15单元中
			
			mov ax,4c00h
			int 21h
code ends

end start
```



# 实验5 编写、调试具有多个段的程序

## 实验任务

一、将下面的程序编译、连接，用Debug加载、跟踪，然后回答问题

```assembly
assume cs:code,ds:data,ss:stack

data segment
	dw 0123h,0456h,0789h,0abch,0defh,0fedh,0cbah,0987h
data ends

stack segment
	dw 0,0,0,0,0,0,0,0
stack ends

code segment
	start:	mov ax,stack
			mov ss,ax
			mov sp,16
			mov ax,data
			mov ds,ax
			push ds:[0]
			push ds:[2]
			
			pop ds:[2]
			pop ds:[0]
			mov ax,4c00h
			int 21h
code ends

end start
```

1) CPU执行程序，程序返回前，data 段中的数据是多少？

首先使用 u 命令查看汇编指令，找出 data 段对应地址，然后使用 d 命令查看内容

{% asset_img Snipaste_2021-09-11_11-13-23.png %}

2. CPU执行程序，程序返回前，cs=076ch，ss=0769h，ds=075ah

{% asset_img Snipaste_2021-09-11_11-15-59.png %}

3. 假设程序加载后，code 段的段地址为 X，则 data 段的段地址为 X-2 ，stack 段的段地址为 X-1

运行发现，data 076AH 、stack 076BH 、code 076CH

{% asset_img Snipaste_2021-09-11_11-21-20.png %}



 

二、将下面的程序编译、连接，用Debug加载、跟踪，然后回答问题

```assembly
assume cs:code,ds:data,ss:stack

data segment
	dw 0123H,0456H
data ends

stack segment 
	dw 0,0
stack ends

code segment 
	start:	mov ax,stack
			mov ss,ax
			mov sp,16
			
			mov ax,data
			mov ds,ax
			
			push ds:[0]
			push ds:[2]
			pop ds:[2]
			pop ds:[0]
			
			mov ax,4c00h
			int 21h
code ends

end start
```

1. CPU执行程序，程序返回前，data 段中的数据是多少？

还是一样，用 u 命令找出 data 段的地址

{% asset_img Snipaste_2021-09-11_11-29-49.png %}

d 命令查看内容

{% asset_img Snipaste_2021-09-11_11-35-03.png %}

2. CPU执行程序，程序返回前，cs=076CH，ss=0769H，ds=075AH

   {% asset_img Snipaste_2021-09-11_11-45-00.png %}

3. 假设程序加载后，code 段的段地址为 X，则 data 段的段地址为 X-2 ，stack 段的段地址为 X-1

4. 对于如下定义的段

```assembly
name segment
···
name ends
```

如果段中的数据占 $N$ 字节，则程序加载后，该段实际占有的空间为 $\left \lceil \frac{N}{16} \right \rceil\times16 $ 字节



三、将下面的程序编译、连接，用Debug加载、跟踪，然后回答问题

```assembly
assume cs:code,ds:data,ss:stack
code segment
	start:	mov ax,stack
			mov ss,ax
			mov sp,16
			mov ax,data
			mov ds,ax
			push ds:[0]
			push ds:[2]
			pop ds:[2]
			pop ds:[0]
			
			mov ax,4c00h
			int 21h
code ends

data segment 
	dw 0123H,0456H 
data ends

stack segment
	dw 0,0
stack ends

end start
```

1. CPU执行程序，程序返回前，data 段中的数据是多少？

{% asset_img Snipaste_2021-09-11_12-13-04.png %}

{% asset_img Snipaste_2021-09-11_12-15-00.png %}

2. CPU执行程序，程序返回前，cs=076AH，ss=0769H，ds=075AH

3. 假设程序加载后，code 段的段地址为 X，则 data 段的段地址为 X+3 ，stack 段的段地址为 X+4

CODE 段地址 076AH，data 段的段地址 076DH ，stack 段的段地址076EH



四、如果将一、二、三题中的最后一条伪指令 `end start` 改为 `end`（也就是说，不指明程序的入口），则哪个程序仍然可以正确执行？请说明原因。

第三个吧，程序一开始就是指令代码，所以能**正常执行**



五、程序如下，编写 code 段中的代码，将 a 段和 b 段中的数据依次相加，将结果存到 c 段中。

```assembly
assume cs:code

a segment
	db 1,2,3,4,5,6,7,8
a ends

b segment
	db 1,2,3,4,5,6,7,8
b ends

c segment
	db 0,0,0,0,0,0,0,0
c ends

code segment
start:	mov ax,a			;answer start
		mov ds,ax
		mov ax,b
		mov es,ax
		
		mov bx,0
		mov cx,8
s:		mov dl,ds:[bx]
		add dl,es:[bx]
		push ds
		mov ax,c
		mov ds,ax
		mov ds:[bx],dl
		pop ds
		inc bx
		loop s
		
		mov ax,4c00h
		int 21h				;answer end		
code ends

end start
```



六、程序如下，编写 code 段中的代码，用 push 指令将 a 段中的前 8 个字型数据，逆序存储到 b 段中

```assembly
assume cs:code
a segment
	dw 1,2,3,4,5,6,7,8,9,0ah,0bh,0ch,0dh,0eh,0fh,0ffh
a ends

b segment
	dw 0,0,0,0,0,0,0,0
b ends

code segment 
	start:	mov ax,a		;answer start
			mov ds,ax
			
			mov ax,b
			mov ss,ax
			mov sp,10h
			
			mov bx,0
			mov cx,8
	s:		push ds:[bx]
			add bx,2
			loop s
			
			mov ax,4c00h
			int 21h			;answer end
code ends

end start
```

