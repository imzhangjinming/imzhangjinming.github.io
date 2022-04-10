---
title: 汇编语言 第十五、十六章 完结撒花
date: 2021-09-25 23:25:58
categories:
- notes
tags:
---
第十五章 外中断

第十六章 直接定址表



<!--more-->



# 第十五章 外中断

## 15.1 接口芯片的端口

> 外设的输入不直接送入内存和CPU，而是送入相关的接口芯片的**端口**中；CPU向外设的输出也不是直接送入外设，而是先送入端口中，再由相关的芯片送到外设。CPU还可以向外设输出控制命令，而这些控制命令也是先送到相关芯片的端口中，然后再由相关的芯片根据命令对外设实施控制。
>
> CPU 通过**端口**和外设进行联系



## 15.2 外中断信息

外设通过**中断信息**来通知CPU端口信息发生变化，需要处理

外设引发的中断信息称为**外中断信息**

引发外中断的信息源称为 **外中断源**，分为以下两类

1. 可屏蔽中断

这是CPU可以**不响应**的外中断

如果标志寄存器 IF = 1 ，则CPU响应中断；如果 IF = 0，则CPU不响应

设置IF位的指令：

`sti` 设置 IF = 1

`cli` 设置 IF = 0

2. 不可屏蔽中断

CPU **必须响应**的外中断

在 8086CPU 中，不可屏蔽中断的中断类型码固定为 2



## 15.3 PC 机键盘的处理过程

1. 键盘输入

> 键盘上的每一个键相当于一个开关，键盘中有一个芯片对键盘上的每一个键的开关状态进行扫描。
>
> 按下一个键时，开关接通，该芯片就产生一个**扫描码**，扫描码说明了按下的键在键盘上的位置。扫描码被送入主板上的相关接口芯片的**寄存器**中，该寄存器的端口地址为 **60h**
>
> 松开按下的键时，也产生一个扫描码，扫描码说明了松开的键在键盘上的位置。松开按键时产生的扫描码也被送入 **60h** 端口中
>
> 一般将**按下**一个键时产生的扫描码称为**通码**，**松开**一个键产生的扫描码称为**断码**。扫描码长度为一个字节，通码的第7位为0，断码的第7位为1，即：
>
> 断码 = 通码 + 80h
>
> {% asset_img Snipaste_2021-09-25_10-22-01.png %}
>
> 续表
>
> {% asset_img Snipaste_2021-09-25_10-22-33.png %}



2. 引发**9号**中断

键盘的输入到达 60h 端口时，相关的芯片就会向 CPU 发出中断类型码为 9 的可屏蔽中断信息。CPU检测到该中断信息后，如果 **IF=1**，则响应中断，引发中断过程，转去执行 int 9 中断例程

3. 执行 int 9 中断例程

> BIOS提供了 int 9 中断例程，用来进行基本的键盘输入处理，主要的工作如下
>
> （1）读出 60h 端口中的扫描码；
>
> （2）如果是**字符键**的扫描码，将该扫描码和它所对应的字符码（即ASCIⅡ码）送入内存中的 BIOS 键盘缓冲区；如果是**控制键**（比如Ctrl）和**切换键**（比如CapsLock）的扫描码，则将其转变为**状态字节**（用二进制位记录控制键和切换键状态的字节）写入内存中存储状态字节的单元；
>
> （3）对键盘系统进行相关的控制，比如说，向相关芯片发出应答信息
>
> 
>
> 0040:17单元存储键盘状态字节，该字节记录了控制键和切换键的状态。键盘状态字节各位记录的信息如下
>
> 0：右shift状态，置1表示按下右shift键；1：左shift状态，置1表示按下左shift键；
> 2：Ctrl状态，置1表示按下Ctrl键；
> 3：Alt状态，置1表示按下Alt键；
> 4：ScrollLock状态，置1表示Scroll指示灯亮；
> 5：NumLock状态，置1表示小键盘输入的是数字；
> 6：CapsLock状态，置1表示输入大写字母；
> 7：Insert状态，置1表示处于删除态。



## 15.4 编写 int 9 中断例程

编程：在屏幕中间依次显示“a”~“z”，并可以让人看清。在显示的过程中，按下 Esc 键后，改变显示的颜色。

```assembly
assume cs:code 

code segment 
start:	mov ax,0b800h 
		mov es,ax 
		mov ah,'a'
s:		mov es:[160*12+40*2],ah 
		call delay
		inc ah 
		cmp ah,'z'
		jna s 
		
		mov ax,4c00h 
		int 21h 
		
delay:	push ax			;delay 子程序用来延时
		push dx
		mov dx,1000h	;进行 10000000h 次空循环
		mov ax,0
sl:		sub ax,1
		sbb dx,0
		cmp ax,0
		jne sl
		cmp dx,0
		jne sl
		pop dx
		pop ax
		ret
code ends
end start
```

上面的程序完成了 在屏幕中间依次显示 “a”~“z” ，每个字母显示一段时间（通过 delay 子程序完成），让人可以看清

那么如何做到每按一次 Esc ，就改变字体颜色呢？

方法就是编写自己的 int 9 中断处理程序并实现以下功能：

（1）从 60h 端口读出键盘的输入；
（2）调用 BIOS 的 int 9 中断例程，处理其他硬件细节；
（3）判断是否为 Esc 的扫描码，如果是，改变显示的颜色后返回；如果不是则直接返回

完整的代码如下

```assembly
assume cs:code

stack segment
	db 128 dup (0)
stack ends

data segment
	dw 0,0
data ends

code segment
start:	mov ax,stack
		mov ss,ax
		mov sp,128
		
		mov ax,data
		mov ds,ax
		
		mov ax,0
		mov es,ax
		
		push es:[9*4]		;这四条代码是存储原来 int 9 中断处理程序的地址
		pop ds:[0]
		push es:[9*4+2]
		pop ds:[2]
		
		mov word ptr es:[9*4],offset int9
		mov es:[9*4+2],cs
		
		mov ax,0b800h
		mov es,ax
		mov ah,'a'
s:		mov es:[160*12+40*2],ah
		call delay
		inc ah
		cmp ah,'z'
		jna s
		
		mov ax,0
		mov es,ax
		
		push ds:[0]			;这四条代码是为了恢复原来的 int 9 中断处理代码地址
		pop es:[9*4]
		push ds:[2]
		pop es:[9*4+2]
		
		mov ax,4c00h
		int 21h
delay:	push ax			;delay 子程序用来延时
		push dx
		mov dx,10h	;进行 100000h 次空循环
		mov ax,0
sl:		sub ax,1
		sbb dx,0
		cmp ax,0
		jne sl
		cmp dx,0
		jne sl
		pop dx
		pop ax
		ret
        
int9:	push ax
		push bx
		push es
		
		in al,60h
		
		;接下来的代码（直到 call dword ptr ds:[0]）是在
		;模拟原来 int 9 中断处理程序的调用过程
		pushf		;标志寄存器入栈
		
		pushf
		pop bx
		and bh,11111100b
		push bx
		popf		;IF=0,TF=0
		call dword ptr ds:[0]		;(IP)=(DS:[0]),(CS)=(DS:[2])
		
		cmp al,1				;判断是不是 ESC 被按下了
		jne int9ret
		
		mov ax,0b800h
		mov es,ax
		inc byte ptr es:[160*12+40*2+1]	;改变字符的颜色属性

int9ret:pop es
		pop bx
		pop ax
		iret
		
code ends
end start
```





# 第十六章 直接定址表

讨论如何有效合理地组织数据

## 16.1 描述了单元长度的标号

```assembly
assume cs:code 
code segment 
	a db 1,2,3,4,5,6,7,8
	b dw 0
start:	mov si,0
		mov cx,8
s:		mov al,a[si]
		mov ah,0
		add b,ax 
		inc si 
		loop s 
		mov ax,4c00h 
		int 21h
code ends
end start
```

注意上面代码中的标号 a 和 b 的后面都没有跟 ':' ，它们是特殊的标号，同时描述内存地址和单元长度

标号 a，描述了地址 code:0，和从这个地址开始，以后的内存单元都是**字节**单元；而标号  b 描述了地址 code:8，和从这个地址开始，以后的内存单元都是**字**单元。

这种标号又叫做**数据标号**



### 检测点 16.1

下面的程序将 code 段中 a 处的 8 个数据累加，结果存储到 b 处的**双字**中，补全程序

```assembly
assume cs:code 
code segment 
	a dw 1,2,3,4,5,6,7,8
	b dd 0
start:	mov si,0
		mov cx,8
s:		mov ax,a[si] 
		add	a[16],ax
		adc a[18],0
		add si,2
		loop s
		
		mov ax,4c00h
		int 21h
code ends
end start
```



## 16.2 在其他段中使用数据标号

一般来说，我们不在代码段中定义数据，而是将数据定义到其他段中。在其他段中，我们也可以使用数据标号来描述存储数据的单元的地址和长度。

> 注意，在后面加有“：”的地址标号，只能在**代码段**中使用，不能在其他段中使用

```assembly
assume cs:code,ds:data 

data segment 
	a db 1,2,3,4,5,6,7,8
	b dw 0
data ends 

code segment 
start:	mov ax,data 
		mov ds,ax 
		mov si,0
		mov cx,8
s:		mov al,a[si]
		mov ah,0
		add b,ax 
		inc si
		1oop s 
		
		mov ax,4c00h 
		int 21h 
code ends
end start
```



注意：如果想在代码段中直接使用**数据标号**访问数据，则需要用 `assume` 把标号所在的**段**和一个**段寄存器**联系起来，比如上面代码中的 `ds:data`



可以将标号当作数据来定义

比如

```assembly
data segment 
	a db 1,2,3,4,5,6,7,8
	b dw 0
	c dw a,b 
data ends
```

就相当于

```assembly
data segment 
	a db 1,2,3,4,5,6,7,8
	b dw 0
	c dw offset a,offset b 
data ends
```



又比如

```assembly
data segment 
	a db 1,2,3,4,5,6,7,8
	b dw 0
	c dd a,b 
data ends
```

相当于

```assembly
data segment 
	a db 1,2,3,4,5,6,7,8
	b dw 0
	c dw offset a,seg a,offset b,seg b 
data ends
```

`seg` 指令取得某一标号的**段地址**



### 检测点 16.2

下面的程序将 data 段中 a 处的 8 个数据累加，结果存储到 b 处的字中，补全程序

```assembly
assume cs:code,es:data 

data segment 
	a db 1,2,3,4,5,6,7,8
	b dw 0
data ends

code segment 
start:	mov ax,data		;answer
		mov es,ax		;answer
		mov si,0
		mov cx,8
s:		mov al,a[si]
		mov ah,0
		add b,ax 
		inc si 
		loop s 
		
		mov ax,4c00h 
		int 21h 
code ends
end start
```



## 16.3 直接定址表

用查表的方法编写程序的技巧



编写子程序，以十六进制的形式在屏幕中间显示给定的字节型数据



分析：显然，我们希望能够在数值 0~15 和字符 “0”~“F” 之间找到一种**映射关系**。这样用 0~15 间的任何数值，都可以通过这种映射关系直接得到“0”~“F”中对应的字符。

具体的做法是，建立一张**表**，表中依次存储字符 “0”~“F”，我们可以通过数值 0~15 直接查找到对应的字符。

程序如下

```assembly
;用al传送要显示的数据

showbyte:	jmp short show 
			table db '0123456789ABCDEF'	;字符表
	show:	push bx
			push es
			
			mov ah,al
			shr ah,1
			shr ah,1
			shr ah,1
			shr ah,1	;右移4位得到高4位的值
			and al,00001111b	;得到低4位的值
			
			mov bl,ah
			mov bh,0
			mov ah,table[bx]
			
			mov bx,0b800h
			mov es,bx
			mov es:[160*12+40*2],ah
			
			mov bl,al
			mov bh,0
			mov ah,table[bx]
			
			mov es:[160*12+41*2],ah
			
			pop es
			pop bx
			
			ret		
```

像这种可以通过依据数据，直接计算出所要找的元素的位置的表，我们称其为直接定址表。



## 16.4 程序入口地址的直接定址表

可以在直接定址表中存放子程序的地址从而方便地实现子程序的调用



实现一个子程序 setscreen ，提供以下功能

（1）清屏：将显存中当前屏幕中的字符设为空格符；
（2）设置前景色：设置显存中当前屏幕中处于奇地址的属性字节的第0、1、2位；

（3）设置背景色：设置显存中当前屏幕中处于奇地址的属性字节的第4、5、6位；
（4）向上滚动一行：依次将第n+1行的内容复制到第n行处；最后一行为空。我们将这4个功能分别写为**4个子程序**，请读者根据编程思想，自行读懂下面的程序。



入口参数说明如下。
（1）用ah寄存器传递功能号：0表示清屏，1表示设置前景色，2表示设置背景色，3表示向上滚动一行；
（2）对于1、2号功能，用al传送颜色值，（al）={0，1，2，3，4，5，6，7}。



```assembly
sub1:	push bx 
		push cx
		push es 
		mov bx,0b800h 
		mov es,bx 
		mov bx,0
		mov cx,2000
sub1s:	mov byte ptr es:[bx],''
		add bx,2
		loop subls 
		
		pop es
		pop cx
		pop bx 
		ret
		
sub2: 	push bx
		push cx
		push es
		
		mov bx,0b800h
		mov es,bx
		mov bx,1
		mov cx,2000
sub2s:	and byte ptr es:[bx],11111000b
		or es:[bx],al
		add bx,2
		loop sub2s
		
		pop es
		pop cx
		pop bx
		ret

sub3:	push bx
		push cx
		push es
		
		mov cl,4
		shl al,cl
		mov bx,0b800h
		mov es,bx
		mov bx,1
		mov cx,2000
sub3s:	and byte ptr es:[bx],10001111b
		or es:[bx],al
		add bx,2
		loop sub3s
		
		pop es
		pop cx
		pop bx
		ret
		
sub4:	push cx
		push si
		push di
		push es
		push ds
		
		mov si,0b800h
		mov es,si
		mov ds,si
		mov si,160
		mov di,0
		cld
		mov cx,24
		
sub4s:	push cx
		mov cx,160
		rep movsb
		pop cx
		loop sub4s
		
		mov cx,80
		mov si,0
sub4s1:	mov byte ptr [160*24+si],''
		add si,2
		loop sub4s1
		
		pop ds
		pop es
		pop di
		pop si
		pop cx
		ret
```



我们可以将这些功能子程序的入口地址存储在一个表中，它们在表中的位置和功能号相对应。对应关系为：功能号*2=对应的功能子程序在地址表中的偏移。程序如下：

```assembly
setscreen:	jmp short set 
			table dw sub1,sub2,sub3,sub4
			
set:		push bx 
			cmp ah,3	;判断功能号是否大于3
			ja sret 
			mov bl,ah 
			mov bh,0
			add bx,bx	;根据ah中的功能号计算对应子程序在tab1e表中的偏移
			ca1l word ptr table[bx]	;调用对应的功能子程序
sret:		pop bx 
			ret
```





第十六章结束啦（实验十五十六没有做，第十七章不看）

汇编语言完结撒花
