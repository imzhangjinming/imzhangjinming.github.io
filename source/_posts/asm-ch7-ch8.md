---
title: 汇编语言 第七、八章 含实验六、七
date: 2021-09-17 21:22:37
categories:
- notes
tags:
---

第七章 更灵活的定位内存地址的方法
第八章 数据处理的两个基本问题



<!--more-->

# 第七章 更灵活的定位内存地址的方法

## 7.1 and 和 or 指令

`and` 按位与，最常用的是将寄存器指定位**置零**

`or` 按位或，最常用的是将寄存器指定位**置一**



## 7.2 关于 ASCII 码



## 7.3 以字符形式给出的数据

我们可以在汇编程序中，用 '….' 的方式指明数据是以字符的形式给出的，编译器将把它们转化为相对应的 ASCII 码

```assembly
assume cs:code,ds:data 
data segment
	db 'unIX'
	db'foRK'
data ends

code segment
	start:	mov al,'a'
			mov bl,'b'

			mov ax,4c00h
			int 21h
code ends

end start
```

`db ‘unIX’`	相当于 `db 75H,6EH,49H,58H`，“u”、“n”、“I”、“X”的ASCII码分别为75H、6EH、49H、58H；
`db 'foRK'`	相当于`db	66H,6FH,52H,4BH`，“f”、“0”、“R”、“K”的ASCII码分别为66H、6FH、52H、4BH；
`mov al,al`	相当于 `mov al,61H`，“a”的ASCII码为61H；
`mov bl,b`	相当于`mov al,62H`，“b”的ASCII码为62H。



## 7.4 大小写转换的问题

观察下列对应大小写字母的ASCII码

{% asset_img Snipaste_2021-09-14_07-06-45.png %}

发现，只要将二进制ASCII码的第5位置一就能确保它是小写字母；置零就能确保是大写字母

置一和置零操作可以用 and 和 or 操作完成



有了前面的铺垫，下面考虑这样一个问题在 codesg 中填写代码，将 datasg 中的第一个字符串转化为大写，第二个字符串转化为小写

```assembly
assume cs:codesg,ds:datasg 
datasg segment
	db 'BaSiC'
	db 'iNfOrMaTion'
datasg ends

codesg segment
start:	mov ax,datasg
		mov ds,ax
		mov bx,0
		mov cx,5
	s:	mov al,[bx]
		and al,11011111B	;转换为大写
		mov [bx],al
		inc bx
		loop s
		
		mov bx,5
		mov cx,11
	s0:	mov al,[bx]
		or al,00100000B		;转换为小写
		mov [bx],al
		inc bx
		loop s0
		
		mov ax,4c00h
		int 21h
		
codesg ends

end start
```



## 7.5 [bx+idata]

`mov ax,[bx+200]`

将一个内存单元的内容送入 ax，这个内存单元的长度为**2个字节**（字单元），存放一个字，偏移地址为 bx 中的**数值**加上200，段地址在   ds 中。

上面那条指令和下面这些指令等价：

`mov ax,[200+bx]`

`mov ax,200[bx]`

`mov ax,[bx].200`



## 7.6  用 [bx+idata] 的方式进行数组的处理

有了 7.5 节的表示方法，我们就可以改写 大小写转换 的程序

```assembly
assume cs:codesg,ds:datasg 
datasg segment
	db 'BaSiC'
	db 'MinIX'
datasg ends

codesg segment
start:	mov ax,datasg
		mov ds,ax
		mov bx,0
		mov cx,5
		
	s:	mov al,0[bx]
		and al, 00100000B	;转换为大写
		mov 0[bx],al
		mov al,5[bx]
		or	al,0010000B		;转换为小写
		mov 5[bx],al
		
		inc bx
		loop s
		
		mov ax,4c00h
		int 21h
		
codesg ends

end start
```



## 7.7 SI 和 DI

si 和 di 是 8086CPU 中和 bx 功能相近的寄存器，si 和 di **不能**够分成两个8位寄存器来使用。下面的3组指令实现了相同的功能。

```ASS
mov bx,0
mov ax,[bx]

mov si,0
mov ax,[si]

mov di,0
mov ax,[di]
```

所以，si、 di 和 ax、bx作用相同，只是 si 和 di **不能**拆分成两个8位寄存器



## 7.8 [bx+si] 和 [bx+di]

又是一种定位内存的方法

`mov ax,[bx+si]`

将一个内存单元的内容送入 ax，这个内存单元的长度为**2字节**（字单元），存放一个字，偏移地址为 bx 中的数值加上 si 中的数值，段地址在 ds 中。

数学化的描述为：**(ax) = ((ds)*16 + (bx) + (si))**



## 7.9 [bx+si+idata] 和 [bx+di+idata]

`mov ax,[bx+si+idata]`

数学化的描述为: (ax) =((ds)*16+(bx)+(si)+idata)



## 7.10 不同寻址方式的灵活运用

比较一下前面用到的几种寻址方式，就可以发现：

（1）[idata] 用一个常量来表示地址，可用于直接定位一个内存单元
（2）[bx] 用一个变量来表示内存地址，可用于间接定位一个内存单元
（3）[bx+idata] 用一个变量和常量表示地址，可在一个起始地址的基础上用变量间接定位一个内存单元
（4）[bx+si] 用两个变量表示地址
（5）[bx+si+idata] 用两个变量和一个常量表示地址



书中从这里开始列举了一系列例子来说明

**例一**

编程，将 datasg 段中每个单词的头一个字母改为大写字母。

```assembly
assume cs: codesg, ds: datasg
datasg segment
	db '1. file         '
	db '2. edit         ' 
	db '3. search       ' 
	db '4. view         '  
	db '5. options      '
	db '6. he1p         ' 
datasg ends

codesg segment 
start: 
codesg ends

end start
```

数据段的数据是一个二维数组，可以用下图来显示：

{% asset_img Snipaste_2021-09-16_08-36-11.png %}

观察到，我们需要改变的是每一行第四列的字母。用两个量进行寻址 bx 标记每一行起始地址，用 [bx+3] 找到要修改的字母，程序如下：

```assembly

assume cs: codesg, ds: datasg
datasg segment
	db '1. file         '
	db '2. edit         ' 
	db '3. search       ' 
	db '4. view         '  
	db '5. options      '
	db '6. he1p         ' 
datasg ends

codesg segment 
start: 	mov ax,datasg
        mov ds,ax
        mov bx,0

        mov cx,6
s:		mov al,[bx+3]
        and al,11011111b
        mov [bx+3],al
        add bx,16
        loop s
codesg ends

end start
```



**例二**

编程，将 datasg 段中**每个单词**改为大写字母

```assembly
assume cs:codesg,ds:datasg 
datasg segment 
	db	'ibm             '
	db	'dec             '
	db 	'vax             '
datasg ends

codesg segment 
start: 
codesg ends 

end start
```



数据段里的数据同样可以用二位数组来表示：

{% asset_img Snipaste_2021-09-16_08-55-52.png %}

只要进行两次循环，外层循环4次行，内层循环3次列，就能更改所有字母

```assembly
mov ax,datasg
mov ds,ax
mov bx,0

mov cx,4
s0:	mov si,0
	
	mov cx,3
s:	mov al,[bx+si]
	and al,11011111b
	mov [bx+si],al
	inc si
	1oop s 
	
	add bx,16
	loop s0
```

但是这个程序有一个问题：内外层循环都用 cx 寄存器计数，执行到内层循环时，cx 内的值就被**覆盖**了，外层循环只能执行一次

解决这个问题也不难，只要在进入内层循环前把 cx 的值保存在另一个寄存器里，结束内循环后再把 cx 的值还原即可

但是这个解决方案也有**问题**：如果程序很复杂，没有多余的寄存器可以使用怎么办呢？

只能使用**内存**了

可以在程序中定义一个空白字，用这块内存存放 cx 中的值

```assembly
assume cs:codesg,ds:datasg 
datasg segment
	db 'ibm             '
	db 'dec             '
	db 'dos             '
	db 'vax             '
	dw 0						;定义一个字，用来暂存 cx
datasg ends

codesg segment
	start:	mov ax,datasg
			mov ds,ax
			mov bx,0
			
			mov cx,4
	s0:		mov ds:[40H],cx		;保存外层循环计数
			mov si,0
			
			mov cx,3
	s:		mov al,[bx+si]
			and al,11011111b
			mov [bx+si],al
			inc si
			loop s
			add bx,16
			mov cx,ds:[40H]		;恢复外层循环计数
			loop s0				;外层循环的 loop 指令将 cx 中的计数值减1
			
			mov ax,4c00H
			int 21H
codesg ends

end start
```



这种方法已经挺好的了，但是如果要保存很多数据，我们就得自己记住每个数据的地址，这比较麻烦。

有没有我们只管存放，程序自动计算地址的东西？有，**栈**就是

```assembly
assume cs:codesg,ds:datasg,ss:stacksg
datasg segment
	db 'ibm             '
	db 'dec             '
	db 'dos             '
	db 'vax             '
datasg ends

stacksg segment
	dw 0,0,0,0,0,0,0,0
stacksg ends

codesg segment
	start:	mov ax,stacksg
			mov ss,ax
			mov sp,10h
			mov ax,datasg
			mov ds,ax
			mov bx,0
			
			mov cx,4
	s0:		push cx
			mov si,0
			
			mov cx,3
	s:		mov al,[bx+si]
			and al,11011111b
			mov [bx+si],al
            inc si
            loop s
            
            add bx,16
            pop cx
            loop s0
            
            mov ax,4c00h
            int 21h
            
codesg ends

end start
```



# 实验六 实践课程中的程序

二、编程，将 datasg 段中的每个单词的**前4个字母**改为大写字母

```assembly
assume cs:codesg, ss:stacksg, ds:datasg 

stacksg segment
	dw	0,0,0,0,0,0,0,0
stacksg ends

datasg segment 
	db'1. display      '
	db'2. brows        '
	db'3. replace      '
	db'4. modify       '
datasg ends

codesg segment
start:	mov ax,stacksg
		mov ss,ax
		mov sp,10h
		mov ax,datasg
		mov ds,ax
		mov bx,0
		
		mov cx,4
	s0:	push cx
		mov si,3
		
		mov cx,4
	s:	mov al,[bx+si]
		and al,11011111b
		mov [bx+si],al
		inc si
		loop s
		
		add bx,16
		pop cx
		loop s0
		
		mov ax,4c00h
		int 21h	
codesg ends

end start
```

执行结果：

{% asset_img Snipaste_2021-09-16_09-49-20.png %}



# 第八章 数据处理的两个基本问题

本章对前面的内容进行总结，所以我会大段摘抄书上的内容

> 我们知道，计算机是进行数据处理、运算的机器，那么有两个基本的问题就包含在其中：
> （1）处理的数据在什么地方？
> （2）要处理的数据有多长？

本章就围绕这两个问题展开



定义两个符号：

reg 表示寄存器（register）

sreg 表示**段**寄存器（segment register）

### reg		  的集合包括：ax、bx、cx、dx、ah、al、bh、bl、ch、cl、dh、dl、sp、bp、si、di

### sreg		的集合包括：ds、ss、cs、es



## 8.1 bx、si、di 和 bp

* 在8086CPU中，**只有 ** bx、si、di 和 bp 这四个寄存器可以用在 `[···]` 中进行寻址，比如：

```assembly
mov ax,[bx]
mov ax,[bx+si]
mov ax,[bx+di]
mov ax,[bp]
mov ax,[bp+si]
mov ax,[bp+di]
```

* 在  `[···]` 中，这四个寄存器可以单独出现，或者只能以4种组合出现：bx和si、bx和di、bp和si、bp和di

```assembly
mov ax,[bx]
mov ax,[si]
mov ax,[di]
mov ax,[bp]
mov ax,[bx+si]
mov ax,[bx+di]
mov ax,[bp+si]
mov ax,[bp+di]
mov ax,[bx+si+idata]
mov ax,[bx+di+idata]
mov ax,[bp+si+idata]
mov ax,[bp+di+idata]
```

* 只要在  `[···]` 中使用寄存器 bp，而指令中没有显性地给出段地址，段地址就**默认在 ss** 中。比如下面的指令

```assembly
mov ax，[bp]				;含义：（ax）=（（ss）*16+（bp））
mov ax，[bp+idata]		;含义：（ax）=（（ss）*16+（bp）+idata）
mov ax，[bp+si]			;含义：（ax）=（（ss）*16+（bp）+（si））
mov ax，[bp+si+idata]	;含义：（ax）=（（ss）*16+（bp）+（si）+idata）
```



## 8.2 机器指令处理的数据在什么地方

机器指令执行之前，所要处理的数据可以在3个地方：CPU内部、内存、端口



## 8.3 汇编语言中数据位置的表达

汇编语言用3个概念来表达数据的位置

* 立即数 idata

对于**直接包含**在机器指令中的数据（执行前在CPU的指令缓冲器中），在汇编语言中称为：立即数（idata），在汇编指令中直接给出

* 寄存器

指令要处理的数据在寄存器中，在汇编指令中给出相应的寄存器名

* 段地址（SA）和偏移地址（EA）



## 8.4 寻址方式

{% asset_img image-20210916204703856.png %}



## 8.5 指令要处理的数据有多长

对于 8086CPU 来说，要处理的数据长度要么是 byte ，要么是 word

所以在机器指令中要指明是 **字操作** 还是 **字节操作**，有以下几种方法：

* 通过寄存器名指明

ax 类的寄存器名指明是 字操作

al 类的寄存器名指明是 字节操作

* 没有寄存器名的情况下，可以使用 `word ptr` 和`byte ptr` 来分别指明字操作和字节操作

用法如下

```assembly
;指明字操作
mov word ptr ds:[0],1
inc word ptr [bx]
inc word ptr ds:[0]
add word ptr [bx],2

;指明字节操作
mov byte ptr ds:[0],1
inc byte ptr [bx]
inc byte ptr ds:[0]
add byte ptr [bx],2
```



## 8.6 寻址方法的综合应用

## 8.7 div 指令

div 是除法指令

* 除数：有8位和16位两种，在一个 reg 或 内存单元中
* 被除数：被除数的存储位置是默认的。如果除数是 8 位，则被除数为16位，默认存放在 AX 中；如果除数是 16位，则被除数为32位，DX 存放高 16 位，AX 存放低16 位
* 结果：如果除数为8位，则商存放在 AL 中，余数存放在 AH 中；如果除数为16位，则商存储在 AX 中，余数存放在 DX 中



## 8.8 伪指令 dd

dd 用来定义 dword（double word）类型数据的，它定义的数据每个占用 4 个字节，两个字

**问题**

用div计算data段中第一个数据除以第二个数据后的结果，商存在第三个数据的存储单元中

```assembly
data segment
	dd 100001
	dw 100
	dw 0
data ends

code segment
	start: 	mov ax,data
			mov ds,ax
			mov ax,ds:[0]
			mov dx,ds:[2]
			mov bx,ds:[4]
			div bx
			mov ds:[6],ax
			
			mov ax,4c00h
			int 21h
code ends

end start
```



## 8.9 dup

和 db、dw、dd等数据定义**伪指令**配合使用，用来进行**数据的重复**

`db 3 dup (0)` 定义了**3**个字节，它们的值都是0，相当于 `db 0,0,0`

`db 3 dup (0,1,2)` 定义了**9**个字节，0、1、2、0、1、2、0、1、2，相当于 `db 0,1,2,0,1,2,0,1,2`

`db 3 dup('abc','ABC')` 定义了**18**个字节，相当于 ` db 'abcABCabcABCabcABC'`



# 实验七 寻址方式在结构化数据访问中的应用

Power idea公司从1975年成立一直到1995年的基本情况如下

| 年份 | 收入（千美元） | 雇员（人） | 人均收入（千美元） |
| :----: | :--------------: | :----------: | :------------------: |
| 1975 | 16 | 3 | ？ |
| 1976 | 22 | 7 | ？ |
| 1976 | 382 | 9 | ？ |
| 1976 | 1356 | 13 | ？ |
| 1976 | 2390 | 28 | ？ |
| 1976 | 8000 | 38 | ？ |
| ··· | | | |
| 1995 | 5937000 | 17800 | ？ |

下面的程序中，已经定义好了这些数据：

```assembly
assume cs:codesg

data segment 
	db	'1975','1976','1977','1978','1979','1980','1981','1982','1983'
	db	'1984','1985','1986','1987','1988','1989','1990','1991','1992'
	db	'1993','1994’,'1995'
	;以上是表示21年的21个字符串
	
	dd 16,22,382,1356,2390,8000,16000,24486,50065,97479,140417,197514
	dd 345980,590827,803530,1183000,1843000,2759000,3753000,4649000,5937000
	;以上是表示21年公司总收入的21个dword型数据

	dw 3,7,9,13,28,38,130,220,476,778,1001,1442,2258,2793,4037,5635,8226
	dw 11542,14430,15257,17800
	;以上是表示21年公司雇员人数的21个word型数据
data ends

table segment 
	db 21 dup('year summ ne ?? ') ;定义了 21x16 个空白字节
table ends

```

编程，将 data 段中的数据按如下格式写入到 table 段中，并计算21年中的人均收入（取整），结果也按照下面的格式保存在 table 段中

{% asset_img Snipaste_2021-09-17_08-12-18.png %}

分析：

年份可以用 table.idata[si] 

其它数据可以用 table.idata

```assembly
assume cs:codesg

data segment 
	db	'1975','1976','1977','1978','1979','1980','1981','1982','1983'
	db	'1984','1985','1986','1987','1988','1989','1990','1991','1992'
	db	'1993','1994','1995'
	;以上是表示21年的21个字符串
	
	dd 16,22,382,1356,2390,8000,16000,24486,50065,97479,140417,197514
	dd 345980,590827,803530,1183000,1843000,2759000,3753000,4649000,5937000
	;以上是表示21年公司总收入的21个dword型数据

	dw 3,7,9,13,28,38,130,220,476,778,1001,1442,2258,2793,4037,5635,8226
	dw 11542,14430,15257,17800
	;以上是表示21年公司雇员人数的21个word型数据
data ends

table segment 
	db 21 dup('year summ ne ?? ') ;定义了 21x16 个空白字节
table ends

stack segment
	dw 0,0
stack ends

codesg segment
	start:	mov ax,data
			mov ds,ax
			mov bx,0
			
			mov ax,table
			mov es,ax
			mov bp,0
			
			mov ax,stack
			mov ss,ax
			mov sp,5
			
			mov di,0
			
			mov cx,21
		s0: push cx
			mov si,0
			
			mov cx,4
		s:	mov ax,[bx][si]
			mov es:[bp].0[si],ax
			
			inc si
			loop s
			
			mov byte ptr es:[bp].4,32
			

			mov ax,84[bx]
			mov es:[bp].5,ax
			mov ax,84[bx+2]
			mov es:[bp].7,ax
			
			mov byte ptr es:[bp].9,32
			
			mov ax,ds:168[di]
			mov es:[bp].10,ax
			
			mov byte ptr es:[bp].12,32
			
			mov ax,es:[bp].5
			;push di
			mov dx,es:[bp].7
			div word ptr es:[bp].10
			mov es:[bp].13,ax
			
			mov byte ptr es:[bp].15,32
			
			;pop di
			pop cx
			add bx,4
			add bp,10h
			add di,2
			loop s0
			
            mov ax,4c00h
            int 21h
codesg ends

end start
```

运行结果：

{% asset_img Snipaste_2021-09-17_12-43-37.png %}

