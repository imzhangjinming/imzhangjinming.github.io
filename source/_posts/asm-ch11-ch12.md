---
title: 汇编语言 第十一、十二章 含检测点和实验十一、十二
date: 2021-09-24 03:27:05
categories:
- notes
tags:
---
第十一章 标志寄存器

第十二章 内中断

<!--more-->

# 第十一章 标志（FLAG）寄存器

flag 和其他寄存器不一样，其他寄存器是用来存放数据的，都是整个寄存器具有一个含义。

flag 寄存器是 **按位** 起作用的，它的每一位都有专门的含义，记录特定的信息

8086CPU 的 flag 寄存器的结构如图11.1所示

{% asset_img Snipaste_2021-09-22_08-55-52.png %}



## 11.1 ZF标志

是 flag 的第六位，**零标志位**。

它记录相关指令执行后，其结果是否为 0。如果结果为 0，那么 zf=1 ；如果结果不为 0，那么 zf=0



> 注意，在 8086CPU 的指令集中，有的指令的执行是**影响标志寄存器**的，比如，`add`、`sub`、`mul`、`div`、`inc`、`or`、`and` 等，它们大都是**运算指令**（进行逻辑或算术运算）；有的指令的执行**对标志寄存器没有影响**，比如，`mov`、`push`、`pop` 等，它们大都是**传送指令**。在使用一条指令的时候，要注意这条**指令**的**全部功能**，其中包括，执行结果对标志寄存器的哪些标志位造成影响。



## 11.2 PF 标志

是 flag 的第二位，**奇偶标志位**

它记录相关指令执行后，其结果的所有 bit 位中 **1 的个数是否为偶数**。如果是偶数，pf=1；如果是奇数，pf=0



## 11.3 SF 标志

是 flag 的第七位，**符号标志位**

它记录相关指令执行后，其结果是否为负；如果为负，sf=1；如果非负，sf=0



### 检测点 11.1

写出下面每条指令执行后，ZF、PF、SF 等标志位的值

```assembly
sub al,al	;ZF=1 PF=1 SF=0
mov al,1	;ZF=1 PF=1 SF=0
push ax		;ZF=1 PF=1 SF=0
pop bx		;ZF=1 PF=1 SF=0
add al,bl	;ZF=0 PF=0 SF=0
add al,10	;ZF=0 PF=1 SF=0
mul al		;ZF=0 PF=1 SF=0
```



## 11.4 CF标志

是 flag 的第零位，**进位标志位**

> 对于位数为N的**无符号数**来说，其对应的二进制信息的最高位，即第N-1位，就是它的最高有效位，而假想存在的第N位，就是相对于最高有效位的更高位，如下图所示
>
> {% asset_img Snipaste_2021-09-22_19-38-16.png %}
>
> 我们知道，当两个数据相加的时候，有可能产生从最高有效位向更高位的进位。比如，两个8位数据：98H+98H，将产生进位。由于这个进位值在8位数中无法保存，我们在前面的课程中，就只是简单地说这个进位值丢失了。其实CPU在运算的时候，并不丢弃这个进位值，而是记录在一个特殊的寄存器的某一位上。8086CPU 就用 flag 的 CF 位来记录这个进位值。比如，下面的指令：
>
> ```assembly
> mov al,98H
> add al,al	;执行后：(al)=30H,CF=1,CF记录了从最高有效位向更高位的进位值
> add al,al	;执行后：(al)=60H,CF=0,CF记录了从最高有效位向更高位的进位值
> ```
>
> 而当两个数据做减法的时候，有可能向更高位借位。比如，两个8位数据：97H-98H，将产生借位，借位后，相当于计算197H-98H。而flag的CF位也可以用来记录这个借位值。比如，下面的指令：
>
> ```assembly
> mov al,97H
> sub al,98H		;执行后:(al)=FFH,CF=1,CF记录了向更高位的借位值
> sub al,al		;执行后:(al)=0,CF=0,CF记录了向更高位的借位值
> ```



## 11.5 OF标志

是 flag 的第十一位，**溢出标志位**

OF 记录了**有符号**数运算的结果是否发生了溢出。如果溢出了，OF=1；没有溢出则OF=0



### 检测点 11.2

写出下面每条指令执行后，ZF、PF、SF、CF、OF等标志位的值

|               | CF   | OF   | SF   | ZF   | PF   |
| :-------------: | :----: | :----: | :----: | :----: | :----: |
| `sub al,al`   | 0 | 0 | 0 | 1 | 1 |
| `mov al,10h`  | 0 | 0 | 0 | 1 | 1 |
| `add al,90h`  | 0 | 0 | 1 | 0 | 1 |
| `mov al,80h`  | 0 | 0 | 1 | 0 | 1 |
| `add al,80h`  | 1 | 1 | 0 | 1 | 1 |
| `mov al,0FCh` | 1 | 1 | 0 | 1 | 1 |
| `add al,05h`  | 1 | 1 | 0 | 0 | 0 |
| `mov al,7Dh`  | 1 | 1 | 0 | 0 | 0 |
| `add al,0Bh`  | 0 | 1 | 1 | 0 | 1 |



## 11.6 adc 指令

**带进位**加法指令，利用了 **CF 位**上记录的进位值

指令格式：`adc 操作对象1,操作对象2`

功能：操作对象1=操作对象1+操作对象2+**CF**

比如指令 `adc ax,bx`实现的功能是:（ax）=（ax）+（bx）+**CF**

CPU提供adc指令的目的，就是来进行**加法的第二步运算**的。adc 指令和 add 指令相配合就可以对**更大的数据**进行加法运算。我们来看一个例子：

编程，计算1EF000H+201000H，结果放在ax（高16位）和bx（低16位）中

因为两个数据的位数**都大于16**，用 add 指令无法进行计算。我们将计算分两步进行，先将低16位相加，然后将高16位和进位值相加。程序如下:

```assembly
mov ax,001EH 
mov bx,0F000H 
add bx,1000H
adc ax,0020H
```



## 11.7 sbb 指令

**带借位**减法指令，利用CF位上记录的借位值

指令格式：`sbb 操作对象1,操作对象2`

功能：操作对象1=操作对象1-操作对象2-CF

比如指令 `sbb ax,bx` 实现的功能是：（ax）=（ax）-（bx）-CF

利用 sbb 指令可以对任意大的数据进行减法运算



## 11.8 cmp 指令

> cmp 是比较指令，cmp 的功能相当于减法指令，只是**不保存结果**。cmp 指令执行后，将对标志寄存器产生影响。其他相关指令通过**识别**这些被影响的**标志寄存器位**来得知比较结果。

cmp 指令格式：`cmp 操作对象1,操作对象2`

功能：计算操作对象1-操作对象2但并**不保存结果**，仅仅根据计算结果**对标志寄存器进行设置**

下面的指令

```assembly
mov ax,8
mov bx,3
cmp ax,bx
```

执行后，(ax)=8，zf=0，pf=1，sf=0，cf=0，of=0



其实，执行 cmp 指令后，我们可以通过分析标志寄存器的值判断两个**操作数的大小关系**

**无符号数的情形**

```assembly
cmp ax,ax
```

如果 $(ax)=(bx)$ 则 $(ax)-(bx)=0$ ，所以：zf=1；

如果 $(ax) \ne (bx)$ 则 $(ax)-(bx)\ne 0$ ，所以：zf=0；

如果 $(ax)<(bx)$ 则 $(ax)-(bx)$ 将产生借位，所以：cf=1；

如果 $(ax)≤(bx)$ 则 $(ax)-(bx)$ 既可能借位，结果可能为0，所以：cf=1 或 zf=1。

如果 $(ax)>(bx)$ 则 $(ax)-(bx)$ 既不必借位，结果又不为0，所以：cf=0 并且 zf=0；

如果 $(ax)≥(bx)$ 则 $(ax)-(bx)$ 不必借位，所以：cf=0；



**有符号数的情形**

```assembly
cmp ah,bh
```

如果 $(ah)=(bh)$ ，则 zf=1

如果 $(ah)\ne(bh)$ ，则 zf=0

如果$(ah)<(bh)$ ，则 (of == 0 && sf == 1)||(of == 1 && sf == 0)

如果 $(ah)>(bh)$ ，则 (of == 0 && sf == 0)||(of == 1 && sf == 1)



## 11.9 检测比较结果的条件转移指令

所有**条件转移指令**的转移位移都是 $[-128，127]$

之前学过的 jcxz 就是一条条件转移指令，它检测的条件是 cx 中的数据是否为 0

除此之外，8086CPU还有其他条件转移指令，它们检测的条件大都**是标志寄存器的相关位**



根据**无符号数**的比较结果进行转移的条件转移指令

| 指令  | 含义         | 检测的相关标志位 |
| :-----: | :------------: | :----------------: |
| `je`  | 等于         | zf=1             |
| `jne` | 不等于则转移 | zf=0             |
| `jb`  | 低于则转移   | cf=1             |
| `jnb` | 不低于则转移 | cf=0             |
| `ja`  | 高于则转移   | cf=0 且 zf=0     |
| `jna` | 不高于则转移 | cf=1 或 zf=1     |



## 检测点 11.3

1. 补全下面的程序，统计 F000:0 处32个字节中，大小在 $[32,128]$ 的数据的个数

```assembly
    mov ax,0f000h 
    mov ds,ax 

    mov bx,0
    mov dx,0
    mov cx,32

s:	mov al,[bx]
	cmp al,32
	jb	s0		;answer
    cmp al,128
    ja	s0		;answer
    inc dx 
s0:	inc bx
    1oops
```

2. 补全下面的程序，统计 F000:0 处32个字节中，大小在 $ (32,128) $ 的数据的个数

```assembly
mov ax,0f000h 
    mov ds,ax 

    mov bx,0
    mov dx,0
    mov cx,32

s:	mov al,[bx]
	cmp al,32
	jna	s0		;answer
    cmp al,128
    jnb	s0		;answer
    inc dx 
s0:	inc bx
    1oops
```



## 11.10 DF标志和串传送指令

DF标志是 flag 的第十位，**方向标志位**。在**串处理**指令中，控制每次操作后 si、di 的增减

df=0每次操作后si、di递增
df=1每次操作后si、di递减

`movsb` 指令

功能：将 ds:si 地址处的一个**字节**转移到 es:di 处。转移后，如果 DF=0，则 si 和 di 加1；如果 DF=1，则 si 和 di 减 1



`movsw` 指令

功能：和 `movsb` 指令类似，只是 `movsw` 一次转移一个**字**。转移后，如果 DF=0，则 si 和 di 加 2；如果 DF=1，则 si 和 di 减 2



上面两个指令搭配 `rep` 指令 (repeat) 就能实现一串数据的传送。比如

`rep movsb`

它的功能就相当于：

```assembly
s:	mov sb
	loop s
```

可见，这条命令会转移 **(cx)** 个字节的数据



使用下面两条指令可以**设置DF的值**

`cld` 将DF置零

`std` 将DF置一



## 11.11 pushf 和 popf

> pushf 的功能是将标志寄存器的值压栈，而 popf 是从栈中弹出数据，送入标志寄存器中。
>
> pushf 和 popf，为直接访问标志寄存器提供了一种方法



### 检测点 11.4

下面的程序执行后：(ax)=？

```assembly
mov ax,0
push ax 
popf 
mov ax,Offf0h 
add ax,0010h 
pushf
pop ax 
and al,11000101B
and ah,00001000B
```

0000 0000 0100 0101B



## 11.12 标志寄存器在 Debug 中的表示

{% asset_img Snipaste_2021-09-23_08-28-49.png %}

我们已知的标志位的表示：

| 标志 | 值为1的标记 | 值为0的标记 |
| :----: | :-----------: | :-----------: |
| of | OV | NV |
| sf | NG | PL |
| zf | ZR | NZ |
| pf | PE | PO |
| cf | CY | NC |
| df | DN | UP |



# 实验十一 编写子程序

编写一个子程序，将包含任意字符，以0结尾的字符串中的小写字母转变成大写字母，描述如下

名称：letterc

功能：将以0结尾的字符串中的小写字母转变成大写字母

参数：ds:si 指向字符串首地址

应用举例：

```assembly
assume cs:codesg

datasg segment
	db "Beginner's All-purpose Symbolic Instruction Code.",0
datasg ends

codesg segment
	begin: 	mov ax,datasg
			mov ds,ax
			mov si,0
			call letterc
			
			mov ax,4c00h
			int 21h
			
	letterc:cmp byte ptr ds:[si],0
			je noletter
			cmp byte ptr ds:[si],97
			jb notlower
			cmp byte ptr ds:[si],122
			ja notlower
			and byte ptr ds:[si],11011111B
notlower:	inc si
			jmp short letterc
noletter:	ret

codesg ends

end begin
```

运行结果：

{% asset_img Snipaste_2021-09-23_08-52-39.png %}





# 第十二章 内中断

这一章主要讲 CPU 内部的中断信息



## 12.1 内中断的产生

对于 8086CPU ，在下面的情况发生的时候会产生相应的中断信息

1. 除法错误
2. 单步执行
3. 执行 into 指令 
4. 执行 int 指令



为了对不同的中断信息进行不同的处理，CPU首先要知道中断信息的来源，也就是识别**中断信息的身份**

识别的操作通过**中断信息的编码**来实现，也就是**中断类型码**



8086CPU 中，中断类型码是一字节的数据

上面所说的四种会产生中断信息的**中断源**的中断类型码分别是：

1. 除法错误：0
2. 单步执行：1
3. 执行 into 指令：4
4. 执行 int 指令，该指令的格式为 int n ，指令中的 n 为字节型数据，是提供给CPU的中断类型码



## 12.2 中断处理程序

CPU的设计者必须在**中断信息**和**它对应的处理程序的入口地址**之间**建立**某种**联系**，使得CPU根据中断信息可以找到要执行的处理程序。



## 12.3 中断向量表

中断向量表就是中断处理程序入口**地址的列表**，CPU 用 8 位的中断类型码通过**中断向量表**找到相应的中断处理程序的**入口地址**

中断向量表保存在**内存**中

8086CPU 中的中断向量表存放在内存的 0000:0000~0000:03FF 的1024个单元中

{% asset_img Snipaste_2021-09-23_09-18-58.png %}

> 那么在中断向量表中，一个表项占多大的空间呢？一个表项存放一个中断向量，也就是一个中断处理程序的入口地址，对于8086CPU，这个入口地址包括段地址和偏移地址，所以一个表项占**两个字**（4 byte），**高地址字**存放**段地址**，低地址字存放偏移地址。



### 检测点 12.1 

1. 用 debug 查看内存，情况如下：

0000:0000 68 10 A7 00 8B 01 70 00-16 00 9D 03 8B 01 70 00

3号中断源处理程序的入口地址为：0070:018B

2. 存储N号中断源处理程序入口的偏移地址的内存单元的地址为：0000:[4N]；存储N号中断源对应的中断处理程序入口的段地址的内存单元地址为 : 0000:[4N+2]



## 12.4 中断过程

> CPU收到中断信息后，要对中断信息进行处理，首先将引发中断过程。硬件在完成中断过程后，CS:IP将指向中断处理程序的入口，CPU开始执行中断处理程序。
>
> 有一个问题需要考虑，CPU在执行完中断处理程序后，应该返回原来的执行点继续执行下面的指令。所以在中断过程中，在**设置CS:IP之前**，还要将**原来的CS和IP的值保存**起来。在使用call 指令调用子程序时有同样的问题，子程序执行后还要返回到原来的执行点继续执行，所以，call指令先保存当前CS和IP的值，然后再设置CS和IP。



8086CPU 引发中断的过程：

1. 获得中断类型码
2. 标志寄存器值入栈
3. 设置标志寄存器 第八位 TF=0 ，第九位 IF=0
4. CS 的内容入栈
5. IP 的内容入栈
6. 从中断向量表获得中断处理程序的入口地址

接下来就开始执行中断处理程序



## 12.5 中断处理程序和 iret 指令

> 中断处理程序的编写方法和子程序的比较相似，下面是常规的步骤:
>
> 1. 保存用到的寄存器
> 2. 处理中断
> 3. 恢复用到的寄存器
> 4. 用 iret 指令返回
>
> iret指令的功能用汇编语法描述为：
>
> ```assembly
> pop IP
> pop CS
> popf
> ```



## 12.6 除法错误中断的处理



## 12.7 编程处理 0 号中断

编程：当发生除法溢出时，在屏幕中间显示 “overflow！”，返回DOS



我们需要做以下几件事情：

1. 编写可以显示 overflow！ 的中断处理程序 do0
2. 将 do0 送入内存 0000:0200 处
3. 将 do0 的入口地址 0000:0200 存储在中断向量表的 0 号表项中



## 12.8 安装中断处理程序的程序

可以使用 `movsb` 指令，将 do0 的代码送入 0:200 处

用 `rep movsb` 指令的时候要确定的信息：

1. 传送的原始位置，段地址：code，偏移地址： offset do0
2. 传送的目的地址，0000:0200
3. 传送的长度：do0 部分代码的长度
4. 传送的方向：正向

```assembly
assume cs:code
code segment
start:	mov ax,cs
		mov ds,ax
		mov si,offset do0					;设置拷贝源地址
		mov ax,0
		mov es,ax
		mov di,0200h						;设置拷贝的目标地址
		
		mov cx,offset do0end-offset do0		;这里使用了一个技巧来计算 do0 代码段的长度
		
		cld									;设置拷贝方向为正向
		rep movsb							;重复执行字节拷贝工作 (cx) 次
		
		设置中断向量表
		
		mov ax,4c00h
		int 21h
do0:	显示字符串“overflow!”
		mov ax,4c00h
		int 21h
do0end:	nop

code ends
end start
```



## 12.9 do0

```assembly
do0:		jmp short do0start
			db "overflow!"
do0start:	mov ax,cs
			mov ds,ax
			mov si,202h
			
			mov ax,0b800h
			mov es,ax
			mov di,12*160+36*2
			
			mov cx,9
s:			mov al,[si]
			mov es:[di],al
			inc si
			add di,2
			loop s
			
			mov ax,4c00h
			int 21h
do0end:		nop
```



## 12.10 设置中断向量

```assembly
mov ax,0
mov es,ax
mov word ptr es:[0] ,200h
mov word ptr es:[2] ,0
```



## 12.11 单步中断

CPU提供单步中断功能的原因是，为单步跟踪程序的执行过程



## 12.12 响应中断的特殊情况

> 一般情况下，CPU在执行完当前指令后，如果检测到中断信息，就响应中断，引发中断过程。可是，在有些情况下，CPU在执行完当前指令后，**即便是发生中断，也不会响应**。



# 实验12 编写0号中断处理函数

编写 0 号中断的处理程序，使得在除法溢出发生时，在屏幕中间显示字符串 ”divide error!“，然后返回 DOS

```assembly
assume cs:codesg

codesg segment
start:		mov ax,0
			mov es,ax
			mov di,200h
			
			mov ax,cs
			mov ds,ax
			mov si,offset do0
			
			mov cx,offset do0end-offset do0
			cld
			rep movsb
			
			mov ax,0
			mov es,ax
			mov word ptr es:[0],200h
			mov word ptr es:[2],0
			
			
			mov ax,1000h
			mov bh,1
			div bh
			
			
			mov ax,4c00h
			int 21h
			
do0:		jmp short do0start
			db "divide error :("

do0start:	mov ax,cs
			mov ds,ax
			mov si,202h
			
			mov ax,0b800h
			mov es,ax
			mov di,160*12+32*2
			
			mov cx,15
s:			mov al,[si]
			mov es:[di],al
			inc si
			add di,2
			loop s
			
			mov ax,4c00h
			int 21h

do0end:		nop

codesg ends

end start
```

运行结果：

{% asset_img Snipaste_2021-09-23_21-40-18.png %}

