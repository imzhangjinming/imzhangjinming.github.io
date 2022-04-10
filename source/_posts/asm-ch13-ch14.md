---
title: 汇编语言 第十三、十四章 含检测点和实验十三、十四
date: 2021-09-25 01:40:22
categories:
- notes
tags:
---
第十三章 int指令

第十四章 端口

<!--more-->



# 第十三章 int 指令

int 指令可以**引发中断**



## 13.1 int 指令

`int n` 

功能：引发 n 号中断



## 13.2 编写供应用程序调用的中断例程

**问题一**：编写、安装中断7ch的中断例程

功能：求一 word 型数据的平方

参数：(ax)=要计算的数据

返回值：dx 存放高16位，ax 存放低16位

应用举例：求 2*3456^2

程序框架如下：

```assembly
assume cs:code 

code segment 
start:	mov ax，3456		;（ax）=3456
		int 7ch			 ;调用中断7ch的中断例程，计算ax中的数据的平方
		add ax，ax 
		adc dx，dx		;dx:ax存放结果，将结果乘以2
		mov ax，4c00h 
		int 21h 
code ends

end start
```



同样的，我们还是要完成以下这些工作：

1. 编写求平方功能的函数
2. 在 0000:0200 处安装程序
3. 设置中断向量表，将程序的入口地址保存在 7ch 表项中，使其成为中断处理程序



安装程序的程序如下：

```assembly
assume cs:code

code segment
start:	mov ax,cs
		mov ds,ax
		mov si,offset sqr
		mov ax,0
		mov es,ax
		mov di,200h
		mov cx,offset sqrend-offset sqr
		cld
		rep movsb
		
		mov ax,0
		mov es,ax
		mov word ptr es:[7ch*4],200h
		mov word ptr es:[7ch*4+2],0
		
		mov ax,4c00h
		int 21h
		
sqr:	mul ax
		iret
		
sqrend:	nop

code ends
end start
```





## 13.3 对 int、iret 和 栈 的深入理解

问题：用 7ch 中断例程完成 loop 指令的功能

应用举例：在屏幕中间显示80个 '!'

```assembly
assume cs:code 
code segment 
start:	mov ax,0b800h 
		mov es,ax 
		mov di,160*12
					
		mov bx,offset s-offset se 		;设置从标号se到标号s的转移位移
		mov cx,80
s:		mov byte ptr es:[di],'!'
		add di,2
		int 7ch							;如果（cx）0，转移到标号s处
se:		nop 

		mov ax,4c00h 
		int 21h 
code ends
end start
```



7ch 中断处理程序如下：

```assembly
1p:		push bp 
        mov bp,sp 
        dec cx 
        jcxz lpret 
        add [bp+2],bx 	;在这里偷偷改变之前入栈的 IP 值，这样调用 iret 时就会自动跳转到 s 处
lpret:	pop bp 
		iret
```



### 检测点13.1

1. 在上面的内容中，我们用 7ch 中断例程实现 loop 的功能，则上面的 7ch 中断例程所能进行的最大转移位移是多少？

16位寄存器能表示的有符号数的范围，-32768~32767

2. 用 7ch 中断例程完成 `jmp near ptr s` 指令的功能，用 bx 向中断例程传送转移位移

应用举例：在屏幕的第12行，显示 data 段中以 0 结尾的字符串

```assembly
assume cs:code 
data segment 
	db 'conversation',0
data ends 

code segment 
start:	mov ax,data
		mov ds，ax 
		mov si，0
		mov ax，0b800h 
		mov es，ax 
		mov di，12*160
s:		cmp byte ptr[si]，0
		je ok						;如果是0跳出循环
		mov al，[si]
		mov es:[di]，al 
		inc si 
		add di，2
		mov bx，offset s-offset ok	;设置从标号ok到标号s的转移位移
		int 7ch						;转移到标号s处
		
ok:		mov ax，4c00h 
		int 21h 
code ends
end start
```



中断例程：

```assembly
jnp:	push bp
		mov bp,sp
		add ss:[bp+2],bx
		pop bp
		iret
```



## 13.4 BIOS 和 DOS 所提供的中断例程

>  在系统板的**ROM**中存放着一套程序，称为BIOS（基本输入输出系统），BIOS中主要包含以下几部分内容：
>
> （1）硬件系统的检测和初始化程序；
> （2）外部中断（第15章中进行讲解）和内部中断的中断例程；
> （3）用于对硬件设备进行I/O操作的中断例程；
> （4）其他和硬件系统相关的中断例程。



## 13.5 BIOS 和 DOS 中断例程的安装过程

> （1）开机后，CPU一加电，初始化 (CS)=0FFFFH，(IP)=0，自动从FFFF:0单元开始执行程序。FFFF:0 处有一条转跳指令，CPU 执行该指令后，转去执行 BIOS 中的硬件系统检测和初始化程序。
> （2）初始化程序将建立 BIOS 所支持的中断向量，即将 BIOS 提供的中断例程的入口地址登记在中断向量表中。注意，对于 BIOS 所提供的中断例程，只需将入口地址登记在中断向量表中即可，因为它们是固化到 ROM 中的程序，一直在内存中存在。
> （3）硬件系统检测和初始化完成后，调用 `int 19h` 进行操作系统的引导。从此将计算机交由操作系统控制。
> （4）DOS启动后，除完成其他工作外，还将它所提供的中断例程装入内存，并建立相应的中断向量。



### 检测点 13.2

判断下面说法的正误：

1. 我们可以编程改变 FFFF:0 处的指令，使得CPU不去执行BIOS中的硬件系统检测和初始化程序

错误。仅仅靠编程（软件）是不行的，还要硬件支持

2. `int 19h` 中断例程，可以由DOS提供

错误。如果 19h 中断例程是由DOS 提供的话，BIOS在完成它自己的工作后就没办法把控制权交给DOS了





## 13.6 BIOS 中断例程应用

设置光标：

```assembly
mov ah,2		;置光标
mov bh,0		;第0页
mov dh,5		;dh中放行号
mov d1,12		;d1中放列号
int 10h
```

显示字符：

```assembly
mov ah,9		;在光标位置显示字符
mov al,'a'		;字符
mov b1,7		;颜色属性
mov bh,0		;第0页
mov cx,3		;字符重复个数
int 10h
```

在屏幕的 5行12列 显示 3 个 **红底高亮闪烁绿色** 的 'a'

```assembly
assume cs:code 
code segment
		mov ah,2		;置光标
		mov bh,0		;第0页
		mov dh,5		;dh中放行号
		mov dl,12		;d1中放列号
		int 10h
        
		mov ah,9			;在光标位置显示字符
		mov al,'a'			;字符
		mov b1,11001010b	;颜色属性
		mov bh,0			;第0页
		mov cx,3			;字符重复个数
		int 10h 
		
		mov ax,4c00h 
		int 21h 
code ends 
end
```



## 13.7 DOS 中断例程应用



# 实验十三 编写、应用中断例程

1. 编写并安装 `int 7ch` 中断例程，功能为显示一个**用 0结束**的字符串，中断例程安装在 0:200 处

参数：(dh) = 行号，(dl) = 列号，(cl) = 颜色，ds:si 指向字符串首地址

```assembly
assume cs:codesg

codesg segment
start:	mov ax,cs
		mov ds,ax
		mov si,offset subcode
		
		mov ax,0
		mov es,ax
		mov di,200h
		
		mov cx,offset subcodeend-offset subcode
		cld
		rep movsb

        mov ax,0
        mov es,ax
        mov word ptr es:[7ch*4],200h
        mov word ptr es:[7ch*4+2],0
		
		mov ax,4c00h
		int 21h
		
subcode:push ax
        push es
        push di
        push si

        mov ax,0b800h
		mov es,ax
        mov al,dh
        mov ah,160
        mul ah
        mov di,ax
        mov al,dl
        mov ah,2
        mul ah
        add di,ax
		
s:		cmp byte ptr [si],0
		je ok
        mov al,[si]
		mov es:[di],al
		inc si
		add di,2
		jmp short s
		
ok:		pop si
        pop di
        pop es
        pop ax
        iret
subcodeend:nop

codesg ends

end start
```

测试程序：

```assembly
assume cs:code 
data segment 
	db "welcome to masm!",0
data ends 

code segment 
	start:	mov dh,10
			mov d1,10
			mov cl,2
			mov ax,data 
			mov ds,ax 
			mov si,0
			int 7ch 
			
			mov ax,4c00h 
			int 21h 
code ends

end start
```

运行结果：

{% asset_img Snipaste_2021-09-24_09-23-54.png %}



3. 下面的程序，分别在屏幕的第2、4、6、8行显示4句英文诗，补全程序

```assembly
assume cs:code 

code segment 
	s1: 	db 'Good, better, best,','$'
	s2: 	db 'Never let it rest,','$'
	s3: 	db 'Till good is better,','$'
	s4:		db 'And better, best.','$'
	s: 		dw offset s1, offset s2, offset s3, offset s4
	row: 	db 2,4,6,8
	
	start:	mov ax,cs 
			mov ds,ax 
			mov bx,offset s 
			mov si,offset row 
			mov cx,4
	ok:		mov bh,0
			mov dh,[si] 		;answer
			mov dl,0
			mov ah,2
			int 10h
			
			mov dx,[bx]			;answer
			mov ah,9
			int 21h
			inc si				;answer
			add bx,2			;answer
			
			loop ok
			mov ax,4c00h
			int 21h

code ends
end start
```

运行结果：

{% asset_img Snipaste_2021-09-24_15-19-15.png %}





# 第十四章 端口

> 在PC机系统中，和CPU通过总线相连的芯片除各种存储器外，还有以下3种芯片
>
> （1）各种接口卡（比如，网卡、显卡）上的接口芯片，它们控制接口卡进行工作；
> （2）主板上的接口芯片，CPU通过它们对部分外设进行访问；
> （3）其他芯片，用来存储相关的系统信息，或进行相关的输入输出处理。
>
> 在这些芯片中，都有一组可以由CPU读写的**寄存器**。这些寄存器，它们在物理上可能处于不同的芯片中，但是它们在以下两点上相同:
>
> （1）都和CPU的总线相连，当然这种连接是通过它们所在的芯片进行的；
> （2）CPU对它们进行读或写的时候都通过控制线向它们所在的芯片发出**端口**读写命令。
>
> 可见，从CPU的角度，将这些**寄存器**都**当作端口**，对它们进行统一编址，从而建立了一个统一的端口地址空间。每一个端口在地址空间中都有一个地址
>
> CPU可以直接读写以下3个地方的数据
>
> （1）CPU内部的寄存器；
> （2）内存单元；
> （3）端口。

操作端口是为了操作不同芯片上的**寄存器**



## 14.1 端口的读写

对端口的读写不能用mov、push、pop等内存读写指令。端口的读写指令只有两条：`in` 和 `out`，分别用于从端口**读取**数据和往端口**写入**数据。



访问端口时总线上的信息

```assembly
in al,60h		;从 60h 号端口读入一个字节
```

① CPU通过**地址**线将地址信息 60h 发出；
② CPU通过**控制线**发出端口读命令，选中端口所在的芯片，并通知它，将要从中读取数据；
③ 端口所在的芯片将 60h 端口中的数据通过**数据线**送入CPU。



> 注意，在 in 和 out 指令中，只能使用 ax 或 al 来存放从端口中读入的数据或要发送到端口中的数据。
>
> 访问**8位**端口时用**al**，访问**16位**端口时用**ax**
>
> 对 0~255 以内的端口进行读写时：
>
> ```ASS 
> in al,20h		;从20h端口读入一个字节
> out 20h,al		;往20h端口写入一个字节
> ```
>
> 对256~65535的端口进行读写时，端口号放在dx中：
>
> ```assembly
> mov dx,3f8h		;将端口号3f8h送入dx 
> in al,dx		;从3f8h端口读入一个字节
> out dx,al		;向3f8h端口写入一个字节
> ```



## 14.2 CMOS RAM 芯片

CMOS RAM 芯片的特征如下：

（1）包含一个**实时钟**和一个有**128个存储单元**的RAM存储器（早期的计算机为64个字节）。
（2）该芯片靠电池供电。所以，关机后其内部的实时钟仍可正常工作，RAM中的信息不丢失。
（3）128个字节的RAM中，内部实时钟占用 0~0dh 单元来保存时间信息，其余大部分单元用于保存系统配置信息，供系统启动时BIOS程序读取。BIOS也提供了相关的程序，使我们可以在开机的时候配置 CMOS RAM 中的系统信息。
（4）该芯片内部有**两个端口**，端口地址为 70h 和 71h。CPU通过这两个端口来读写 CMOS RAM
（5）70h 为地址端口，存放**要访问的** CMOS RAM单元的地址；71h 为数据端口，存放从选定的 CMOS RAM 单元中读取的数据，或要写入到其中的数据。可见，CPU对 CMOS RAM 的读写分两步进行，比如，读 CMOS RAM 的2号单元：

① 将 2 送入端口 70h；
② 从端口 71h 读出2号单元的内容。



### 检测点 14.1

1. 编程，读取 CMOS RAM 的2号单元的内容

```assembly
mov al,2
out 70h,al
in al,71h
```

2. 编程，向 CMOS RAM 的 2 号单元写入 0

```assembly
mov al,0
out 71h,al
out 70h,2
```



## 14.3 shl 和 shr 指令

`shl` 逻辑左移，功能是：

（1）将一个寄存器或内存单元中的数据向左移位；
（2）将**最后移出**的一位**写入CF**中；
（3）最低位用0补充。



如果移动的**位数大于1**，必须将移动的位数放在 cl 中

```assembly
mov al,01010001b 
mov cl,3
shl al,cl
```



`shr` 是逻辑右移指令，它和 `shl` 所进行的操作刚好相反

（1）将一个寄存器或内存单元中的数据向右移位；
（2）将最后移出的一位**写入CF**中；
（3）**最高位**用**0**补充。



同样的，如果移动的**位数大于1**，必须将移动的位数放在 cl 中



### 检测点 14.2

编程，用加法和移位指令计算 (ax)=(ax)*10

```assembly
mov bx,ax
shl bx,1
mov ax,bx
mov cl,2
shl bx,cl
add ax,bx
```



## 14.4 CMOS RAM 中存储的时间信息

这一节全部摘抄

> 在 CMOS RAM 中，存放着当前的时间：年、月、日、时、分、秒。这 6 个信息的长度都为 1 个字节，存放单元为：
>
> 秒：0	分：2	时：4	日：7	月：8	年：9
>
> 这些数据以BCD码的形式存放
>
> {% asset_img Snipaste_2021-09-24_20-14-36.png %}



编程，在屏幕中间显示当前的月份

```assembly
assume cs:code
code segment
start:	mov al,8
		out 70h,al
		in al,71h
		
		mov ah,al
		mov cl,4
		shr ah,cl
		and al,00001111B
		
		add ah,30h
		add al,30h
		
		mov bx,0b800h
		mov es,bx
		mov byte ptr es:[160*12+40*2],ah
		mov byte ptr es:[160*12+41*2],al
		
		mov ax,4c00h
		int 21h
		
code ends
end start
```



# 实验 十四 访问 CMOS RAM

编程，以 ”年/月/日 时:分:秒“  的格式显示当前的日期和时间

我打算一个一个字符写入

```assembly
assume cs:codesg

codesg segment
start:	mov ax,0b800h
		mov es,ax
		
		mov al,9
		out 70h,al
		in al,71h
		
		mov ah,al
		mov cl,4
		shr ah,cl
		and al,00001111B
		
		add ah,30h
		add al,30h

		mov byte ptr es:[160*12],ah
		mov byte ptr es:[160*12+1*2],al
		mov byte ptr es:[160*12+2*2],'/'
		
		mov al,8
		out 70h,al
		in al,71h
		
		mov ah,al
		mov cl,4
		shr ah,cl
		and al,00001111B

		add ah,30h
		add al,30h		
	
    	mov byte ptr es:[160*12+3*2],ah
		mov byte ptr es:[160*12+4*2],al
		mov byte ptr es:[160*12+5*2],'/'
		
		mov al,7
		out 70h,al
		in al,71h
		
		mov ah,al
		mov cl,4
		shr ah,cl
		and al,00001111B

		add ah,30h
		add al,30h

		mov byte ptr es:[160*12+6*2],ah
		mov byte ptr es:[160*12+7*2],al
		mov byte ptr es:[160*12+8*2],' '
		
		mov al,4
		out 70h,al
		in al,71h
		
		mov ah,al
		mov cl,4
		shr ah,cl
		and al,00001111B
		
        add ah,30h
		add al,30h		
		
        mov byte ptr es:[160*12+9*2],ah
		mov byte ptr es:[160*12+10*2],al
		mov byte ptr es:[160*12+11*2],':'
		
		mov al,2
		out 70h,al
		in al,71h
		
		mov ah,al
		mov cl,4
		shr ah,cl
		and al,00001111B

		add ah,30h
		add al,30h

		mov byte ptr es:[160*12+12*2],ah
		mov byte ptr es:[160*12+13*2],al
		mov byte ptr es:[160*12+14*2],':'
		
		mov al,0
		out 70h,al
		in al,71h
		
		mov ah,al
		mov cl,4
		shr ah,cl
		and al,00001111B

		add ah,30h
		add al,30h
        		
		mov byte ptr es:[160*12+15*2],ah
		mov byte ptr es:[160*12+16*2],al
		;mov byte ptr es:[160*12+14*2],':'
		
		mov ax,4c00h
		int 21h
		
codesg ends

end start		
```

运行结果：

{% asset_img Snipaste_2021-09-24_20-38-05.png %}

