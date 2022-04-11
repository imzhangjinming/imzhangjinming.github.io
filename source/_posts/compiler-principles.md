---
title: 国防科大编译原理课程观光游记
date: 2022-04-11 21:55:08
categories:
- notes
tags:
---
课程视频地址：

[国防科技大学-编译原理（国家级精品课）高清流畅](https://www.bilibili.com/video/BV11t411V74n)

<!--more-->

# 第二章 高级语言及其语法描述

 ## 2.1 程序语言的定义

### 语法：一组规则，在它的规范下，一组字符串可以成为一个 well-formed 的程序

* 词法规则：单词符号的形成规则
	* 一般包括：常数、标识符、基本字等 
	* 描述工具：有限自动机
* 语法规则：语法单位的形成规则
	* 语法单位通常包括：表达式、语句、函数等
	* 描述工具：上下文无关文法

### 语义：一组规则，用来定义程序的意义

* 语义的描述方法
	* 自然语言描述
	* 形式描述
		* 操作语义
		* 指称语义
		* 代数语义

## 2.2 高级语言的一般特性

1.高级语言的分类

* 强制式语言 Imperative Language
	* FORTRAN、C
* 应用式语言 Applicative Language
* 基于规则的语言 Rule-based Language
* 面向对象语言 Object-oriented Language



2.数据类型和操作

一种数据类型应该具有的特点：

* 用于区别该数据类型的**属性**
* 可以施加到该数据类型上的**操作**
* 数据类型可以有的**值**



## 2.3 程序语言的语法描述

一些名词术语：

* 字母表$\Sigma$ :一个**有穷**字符集，其中的每个元素称为一个字符
* $\Sigma$ 上的**字** ：由$\Sigma$中的字符构成的有穷序列
* 空字$\varepsilon$ : 不包含任何字符的字
* $\Sigma^{*}$ ： 字母表$\Sigma$上的**所有字**的全体，包含空字
* $\Sigma^{*}$ 的子集 $U$ 和 $V$ 的**积**定义为

$$
UV = \left\{\ \alpha\beta\ \mid\ \alpha\in U\ \&\ \beta \in V\right\}
$$

* $V$ 自身的 n 次积记为 $V^{n}$
* $V^{0}=\left\{ \varepsilon \right\}$

* $V$ 的**闭包**： $V^{*}=V^{0}\cup V^{1}\cup V^{2}\cup V^{3}\cup V^{4}\dots$ 

* $V$的**正规闭包**： $V^{+}=VV^{*}$

这里有一个小注意点：如果$V$中本身不含有空字$\varepsilon$，那么$V$的正规闭包中也不含有空字，但是$V$的闭包中有空字；其他情况下，闭包与正规闭包等价。



### 上下文无关文法

文法：描述语言语法结构的规则

一个上下文无关文法$G$是一个**四元式**
$$
G=(V_{T},\ V_{N},\ S,\ P)
$$

* $V_{T}$ : 终结符集合（非空）
* $V_{N}$ : 非终结符集合（非空），$V_{N}$ 与 $V_{T}$ 交集为空 
* $S$ : 文法的开始符，$S \in V_{N}$

* $P$ : 产生式的集合（有限）
* 开始符 $S$ 至少在**某个产生式的左边**出现一次



**定义**：称 $\alpha A \beta$ 直接推出 $\alpha \gamma \beta$, 即
$$
\alpha A \beta \Rightarrow \alpha \gamma \beta
$$
仅当$A \rightarrow \gamma$ 是一个产生式，且$\alpha, \beta \in (V_{T}\ \cup\ V_N)^*$。

通常，用 $\alpha_{1}\ \sideset{}{}\Rightarrow^{+}\ \alpha_{n}$ 表示 $\alpha_1$ 经过**一次或多次**推导得到$\alpha_n$；  $\alpha_{1}\ \sideset{}{}\Rightarrow^{*}\ \alpha_{n}$ 表示 $\alpha_1$ 经过**零次或多次**推导得到$\alpha_n$ 。

这里的$+$ 和 $*$ 的含义与它们在正则表达式中的含义相同。



**定义**：假定$G$是一个文法，$S$是它的开始符号。如果$S \Rightarrow^{*} \alpha$，则称$\alpha$为一个句型。仅含**终结符**的句型是一个**句子**。文法$G$产生的句子全体是一个**语言**，将它记为$L(G)$
$$
L(G)=\{\alpha \mid\ S\Rightarrow^{+}\alpha ,\ \alpha \in V_T^*\}
$$
**最左推导**：任何一步 $\alpha \Rightarrow \beta$ 都是对$\alpha$中最左边的非终结符进行替换

**最右推导**：任何一步 $\alpha \Rightarrow \beta$ 都是对$\alpha$中最右边的非终结符进行替换



### 语法树与二义性

**定义**：如果一个文法中的某个句子的语法树不唯一，那么这个文法是二义的

**语言的二义性**：对于一种语言，如果找不出对应的无二义性的文法，那么这个语言就是二义性的



巴克斯范式 BNF



# 第三章 词法分析

* 词法分析的**任务**：从左到右逐个字符地对源程序进行扫描，产生一个个单词符号

* 词法分析器（Lexical Analyzer）又称扫描器（Scanner）

## 3.1 对于词法分析器的要求

功能：输入源程序，输出单词符号

* 单词符号的种类：
	* 基本字：语言内置关键字
	* 标识符：各种变量名、函数名等
	* 常数
	* 运算符：+ - * / 等
	* 界符：逗号、分号、括号、空白等

* 输出的单词符号的表示形式

$$
(单词种别，单词自身的值)
$$

单词种别标识符号的类别，实践中通常会给不同类别**编码**



## 3.2 词法分析器的设计

一般结构

{% asset_img Snipaste_2022-02-12_14-44-16.png %}





### 状态转换图：帮助词法分析的工具

{% asset_img Snipaste_2022-02-12_15-09-55.png %}

状态转换图的具体实现：

* 不含回路的分叉节点

	可以用一个`switch`语句或者一组`if-then-else`语句实现

	{% asset_img Snipaste_2022-02-12_15-20-58.png %}

* 含有回路的状态节点

	可以用`while`语句实现

	{% asset_img Snipaste_2022-02-12_15-21-50.png %}

* 终态节点

	表示已经识别出了某种单词符号，返回并做一些处理即可

	`return (C, VAL)` 其中C是单词种别， VAL为单词自身的值

	{% asset_img Snipaste_2022-02-12_15-24-32.png %}

## 3.3 正规表达式与有限自动机

### 3.3.1 正规式和正规集

* 正规集可以用正规表达式表示
* 正规表达式是表示正规集的一种方法
* 一个字集合是**正规集**当且仅当它能用正规式表示



**正规式和正规集的递归定义**

对于给定的字母表$\Sigma$

* 任何$a \in \Sigma$，$a$ 是$\Sigma$上的正规式，它表示的正规集为$\{a\}$
* $\varepsilon$ 和 $\varnothing$ 都是 $\Sigma$上的正规式，它们所表示的正规集为 $\{\varepsilon\}$ 和 $\varnothing$

* 假定 $e_1$ 和 $e_2$ 都是 $\Sigma$ 上的正规式，它们所表示的正规集为 $L(e_1)$ 和 $L(e_2)$，则
	* $(e_1\mid e_2)$ 为正规式，它表示的正规集为  $L(e_1)\cup L(e_2)$
	* $(e_1 \cdot e_2)$ 为正规式，它表示的正规集为  $L(e_1)L(e_2)$
	* $(e_1)^*$ 为正规式，它表示的正规集为  $(L(e_1))^*$

仅由**有限次**使用上面三个步骤而定义的表达式才是$\Sigma$上的正规式，仅由这些正规式表示的**字集**才是$\Sigma$上的正规集



### 3.3.2 确定有限自动机（Deterministic Finite Automata, DFA）

确定有限自动机$M$是一个五元式：
$$
M=(S,\ \Sigma,\ f,\ S_0,\ F)
$$
其中：

* $S$：有穷状态集
* $\Sigma$：有穷输入字母表 （正是状态集和字母表的有穷性体现了DFA**有限**的特性）
* $f$：状态转换函数
* $S_0 \in S$：**唯一**的初态
* $F \subseteq S$: 终态的**集合**

⚠️注意：**初态**必须有，且是**唯一**的；而**终态**可以有**0个**，**1个**或**多个**，它们的集合就是 $F$





{% asset_img Snipaste_2022-02-12_20-44-26.png %}



### 3.3.3 非确定有限自动机（Nondeterministic Finite Automata, NFA）

非确定有限自动机$M$是一个五元式：
$$
M=(S,\ \Sigma,\ f,\ S_0,\ F)
$$
其中：

* $S$：有穷状态集
* $\Sigma$：有穷输入字母表 （正是状态集和字母表的有穷性体现了NFA**有限**的特性）
* $f$：状态转换函数，为 $S \times \Sigma^* \rightarrow 2^S$ 的部分映射（这里好像涉及到集合论的知识，我不明白）
* $S_0 \subseteq S$：**非空**的初态**集**
* $F \subseteq S$: **可空**的终态**集合**



* NFA 与 DFA 的区别（从状态转换图来看）

	{% asset_img Snipaste_2022-02-12_21-07-05.png %}

	* NFA 可以有**多个初态**
	* NFA 弧上的标记可以是$\Sigma^*$中的一个**字**，不一定是单个字符

	* 同一个字可能出现在同状态射出的多条弧上



* 对于$\Sigma^*$中的任何字$\alpha$，如果存在一条从初态到某一终态的道路，且这条路上所有弧上的字连接成的字等于$\alpha$，则称$\alpha$为NFA M所识别（也就是这个自动机能够识别$\alpha$这个字）

* NFA M所识别的字的全体称为 $L(M)$



NFA 更加直观，易于人工设计，而 DFA 则更容易编程实现。

所以课上这里证明了：任何一个 NFA 都可以转化为等价的 DFA。

这就在**NFA 和 DFA 之间架起了一座桥梁**。



### 3.3.4 正规文法与有限自动机的等价性

* 等价的含义：对于正规文法$G$和有限自动机$M$，如果$L(G)=L(M)$，则称$G$ 和 $M$ 是等价的
* 关于正规文法和有限自动机等价性，有如下结论：
	* 对每一个右线性正规文法$G$或左线性正规文法$G$，都存在一个有限自动机（FA） $M$，使得$L(M)=L(G)$
	* 对每一个 FA $M$，都存在一个右线性正规文法$G_R$ 和左线性正规文法$G_L$，使得$L(G_R)=L(G_L)=L(M)$

这样在**正规文法和有限自动机之间架起了一座桥梁**



### 3.3.5 正规式与有限自动机的等价性

**定理**：

* 对任何有限自动机（FA）M，都存在一个正规式$r$，使得$L(r)=L(M)$
* 对任何正规式$r$，都存在一个FA M，使得$L(M)=L(r)$

这样就在**正规式和 FA 之间架起了又一座桥梁**。



### 3.3.6 确定有限自动机（DFA）的化简

* 对 DFA $M$ 的化简：寻找一个**状态数**比M少的 DFA $M^{'}$，使得 $L(M)=L(M^{'})$

* 假设$s$ 和 $t$ 为 $M$ 的两个状态，称 $s$ 和 $t$ 等价：如果从状态$s$出发能读出某个（任意）字$\alpha$而停止于终态，那么同样，从$t$出发也能读出$\alpha$而停止于终态；反之亦然
* 假设$s$ 和 $t$ 为 $M$ 的两个状态，称 $s$ 和 $t$ **不**等价：**存在一个字**$\alpha$，如果从状态$s$出发能读出字$\alpha$而停止于终态，从$t$出发读出$\alpha$而停止于**非**终态；或者如果从状态$t$出发能读出字$\alpha$而停止于终态，从$s$出发读出$\alpha$而停止于**非**终态
* 两个状态不等价，则称它们是**可区分**的



**DFA M 最少化的基本思想**

把$M$的状态集划分为一些不相交的子集，使得任何两个不同子集的状态是可区分的，而同一子集的任何两个状态是等价的。

最后在每个子集里选出一个代表，同时消去其他状态。



## 3.4 小结

第三章建立了这样一个结构：

{% asset_img Snipaste_2022-02-13_10-36-21.png %}

首先，在3.3.1小节中，我们认识了正规式、正规集以及它们之间的关系。

在3.3.2和3.3.3小节中，我认识了DFA、NFA以及它们之间的关系：DFA是NFA的特例，但是DFA和NFA的表述能力相同，所有NFA都可以确定化为DFA。

一般来说，DFA易于编程实现，但是不便于人类理解；NFA易于人工设计。所以词法分析的主要思路是：

* 从正规式或正规文法得到对应的NFA
* 将NFA确定化为DFA
* 如果需要，先化简DFA。然后编程实现DFA，得到词法分析程序



**LEX**：词法分析器产生器



# 第四章 语法分析——自上而下分析

## 4.1 语法分析器的功能

* 语法分析器的任务
	* 分析一个文法的句子结构
* 语法分析器的功能
	* 按照文法的产生式（语言的语法规则），识别输入符号串是否为一个句子（合式程序）

* 语法分析的方法
	* 自下而上分析法（Bottom-up）
	* 自上而下分析法（Top-down）
		* 基本思想
			* 从文法开始符号出发，反复使用各种产生式，寻找“匹配”的推导
		* 递归下降分析方法
			* 对每一个语法变量（非终结符）构造一个相应的子程序，每个子程序识别一定对语法单位
			* 通过子程序之间的相互调用实现对输入串的识别
		* 预测分析程序



## 4.2 自上而下分析面临的问题

* 自上而下就是从文法的开始符号出发，向下推导，推出句子
* 自上而下分析的主旨
	* 针对输入串，从开始符号（根节点）出发，自上而下地为输入串建立**语法树**
	* 核心就是寻找输入串的最左推导

* 自上而下面临的问题
	* 回溯
	* 左递归文法无限循环



## 4.3 LL(1) 分析法

解决 4.2 中的两个问题：

### 4.3.1 消除文法的左递归性：改写为等价的右递归文法

假定$P$的全部产生式是：
$$
P \rightarrow P\alpha_1 \mid P\alpha_2 \mid \cdots \mid P\alpha_m \mid \beta_1 \mid \beta_2 \mid \cdots \mid \beta_n
$$
其中每个$\alpha$不等于$\varepsilon$，每个$\beta$都不以$P$开头

下面是将上述左递归文法改写为等价的右递归文法的式子：
$$
P \rightarrow \beta_1 P^{'} \mid \beta_2  P^{'}\mid \cdots \mid \beta_n P^{'} \newline
P^{'} \rightarrow \alpha_1 P^{'} \mid \alpha_2 P^{'} \mid \cdots \mid \alpha_m P^{'} \mid \varepsilon 
$$


### 4.3.2 消除回溯、提左因子

为了消除回溯必须保证：对文法的任何非终结符，当要用它去匹配输入串时，能够根据它所面临的输入符号准确地指派它的一个候选去执行任务

**终结首符集的定义**

令$G$是一个不含左递归的文法，对$G$的所有非终结符的每个候选$\alpha$，定义它的**终结首符集**$First(\alpha)$为
$$
FIRST(\alpha)=\{ a\mid \alpha \Rightarrow^* a...\ ,\ a\in V_T\}
$$
特别地，如果$\alpha \Rightarrow^* \varepsilon$，则$\varepsilon \in FIRST(\alpha)$



如果非终结符$A$的所有候选的终结首符集两两不相交，即$A$的任何两个不同候选$\alpha_i$ 和 $\alpha_j$
$$
FIRST(\alpha_i)\cap FIRST(\alpha_j)=\varnothing
$$
当要求$A$匹配输入串时，$A$就能根据它所面临的第一个输入符号$a$，准确地指派某一个候选去执行任务。这个候选就是终结首符集含有$a$的$\alpha$



有了这个定义后，描述消除回溯问题就简洁很多了。现在还有一个问题，如果一个非终结符的多个候选都有同样的开头，比如
$$
A \rightarrow \delta\beta_1\mid \delta\beta_2\mid \cdots \mid \delta\beta_n\mid \gamma_1\mid \gamma_2\mid \cdots \mid \gamma_m
$$
那么$A$的前n个候选都有同样的终结首符集，怎么办呢？

**提取公共左因子**

可以把$A$的规则改写如下
$$
A \rightarrow \delta A^{'}\mid  \gamma_1\mid \gamma_2\mid \cdots \mid \gamma_m \newline
A^{'} \rightarrow \beta_1\mid \beta_2\mid \cdots \mid \beta_n
$$
这样$A$的候选的所有终结首符集就不相交了。

经过反复提取左因子，就能够把每个非终结符（包括新引进的非终结符，比如上面的$A^{'}$）的所有候选首符集变成两两不相交的



### 4.3.3 LL(1) 分析条件

假定$S$是文法$G$的开始符号，对于$G$的任何非终结符$A$，定义$A$的$FOLLOW$集合
$$
FOLLOW(A)=\{ a \mid S\Rightarrow^*...Aa...,\ a \in V_T\}
$$
特别地，如果$S\Rightarrow^* \cdots A$，则规定$\# \in FOLLOW(A)$



**构造不带回溯的自上而下分析的文法条件**

* 文法不含递归
* 对于文法中的每一个非终结符$A$的各个产生式的候选首符集两两不相交：

$$
\begin{align*}
& 若\ A \rightarrow \alpha_1 \mid \alpha_2 \mid \cdots \mid \alpha_n \ ,\newline
& 则\ FIRST(\alpha_i) \cap FIRST(\alpha_j)=\varnothing\ (i \ne j)
\end{align*}
$$

* 对文法中的每个非终结符$A$，若它存在某个候选首符集包含$\varepsilon$，则须

$$
FIRST(\alpha_i)\cap FOLLOW(A)=\varnothing,\ i=1,2,...,n
$$

如果一个文法$G$满足以上条件，则称该文法为LL(1)文法

*LL(1) 中各个字符的含义：L-从左到右扫描字符串；L-最左推导；1-分析时每一步只需要向前查看一个符号*

{% asset_img Snipaste_2022-02-13_17-37-23.png %}





#### 如何构造FIRST和FOLLOW集合？

直接贴PPT

* 构造**每个文法符号**的**FIRST()**集合

{% asset_img Snipaste_2022-02-13_18-39-17.png %}

构造**每个文法符号**的**FIRST()**集合续

{% asset_img Snipaste_2022-02-13_18-41-10.png %}



* 构造**任何符号串**的**FIRST()**集合

{% asset_img Snipaste_2022-02-13_18-58-35.png %}





* 构造每个非终结符的**FOLLOW()**集合

	对于文法$G$的每个非终结符$A$构造$FOLLOW(A)$的办法是，连续使用下面的规则，直至每个$FOLLOW$不再增大为止：

	* 对于开始符号$S$，置$\#$于$FOLLOW(S)$中；
	* 如果$A\rightarrow \alpha B \beta$是一个产生式，则把$FIRST(\beta)\setminus \{\varepsilon \}$ 加到 $FOLLOW(B)$ 中；
	* 如果$A\rightarrow \alpha B$是一个产生式，或者$A\rightarrow \alpha B \beta$是一个产生式且$\beta \Rightarrow^* \varepsilon$，则把$FOLLOW(A)$加到$FOLLOW(B)$中。



## 4.4 递归下降分析程序构造

**主要思路**

* 分析程序由一组递归过程组成，对每一语法变量（非终结符）构造一个相应的子程序，识别相应的语法单位
* 通过子程序之间的**相互调用**实现对输入串的识别
* 这样的分析程序称为**递归下降分析器**





**文法的另一种表示法和转换图**

在元符号 $\rightarrow$ 和 $\mid$ 的基础上，扩充几个元语言符号：

* 用花括号$\{a\}$表示闭包运算 $a^*$
* $\{a\}_0^n$表示 $a$ 可任意重复 0 到 n 次
* $[a]$表示$\{a\}_0^1$，即$a$可有可无

引入了上述元符号的文法称为扩充的巴克斯范式



## 4.5 预测分析程序：递归下降的非递归实现

预测分析程序的构成：

{% asset_img Snipaste_2022-02-14_09-21-10.png %}

* 总控程序：根据**栈顶符号**和**当前输入符号**执行动作
* 分析表：$M[A,\ a]$矩阵，$A\in V_N ,\ a \in V_T$

* 分析栈STACK : 用于存放文法符号 



#### 分析开始时：

栈顶是开始符号$S$，输入指针指向输入单词串的首个单词

{% asset_img Snipaste_2022-02-14_09-27-49.png %}



#### 分析的中间过程：

{% asset_img Snipaste_2022-02-14_09-30-11.png %}

根据栈顶符号$X$和当前输入符号$a$，总控程序执行一下三种动作之一

* 如果 $X=a='\#'$ ，则分析成功，停止分析
* 如果 $X=a \ne '\#'$ ，则把$X$从栈中弹出，让$a$指向下一个输入符号 （意味着这一个终结符匹配成功了）
* 如果$X$是一个非终结符，则需要查看分析表$M$
	* 若$M[X,a]$中存放着关于$X$的一个产生式，则把$X$弹出栈并把产生式的右部符号串按**反序**进栈
	* 若$M[X,a]$中存放着错误标志，则调用报错程序



#### 分析表$M[A,a]$的构造

在已经构造出每个非终结符$A$及其任意候选$\alpha$的$FIRST(\alpha)$和$FOLLOW(A)$的基础上，构造文法$G$的分析表$M[A,\ a]$ :

* 对文法$G$的每一个产生式$A \rightarrow \alpha$执行：
	* 对每个终结符$a \in FIRST(\alpha)$，把$A \rightarrow \alpha$ 加到$M[A,\ a]$中
	* 如果$\varepsilon \in FIRST(\alpha)$，则对任何$b\in FOLLOW(A)$把$A\rightarrow \alpha$加到$M[A,\ b]$中

* 把所有无定义的$M[A,\ a]$标上出错标志



# 第五章 语法分析——自下而上分析

## 5.1 自下而上分析的基本问题

#### 基本思想

* 从输入串开始，逐步进行规约，直到文法的**开始符号**
* 规约：根据文法的产生式规则，把产生式**右部**替换成**左部**
* 从树的叶子节点开始构造语法树



#### 算符优先分析法

* 按照算符的优先关系和结合性质进行语法分析
* 适合分析**表达式**



#### 移进-规约

基本思想：用一个寄存符号栈，把输入符号一个个地移进到栈里，当栈顶形成某个产生式的**候选式**时，就把栈顶的这一部分替换成该产生式的左部符号

移进-规约思想的核心是识别**可规约串**



**短语、直接短语和句柄的定义**

令$G$是一个文法，$S$是文法的开始符号，假定$\alpha \beta \delta$是文法$G$的一个句型，如果有
$$
S\Rightarrow^* \alpha A \delta\ 且\ A \Rightarrow^+ \beta
$$
则称$\beta$是句型$\alpha \beta \delta$相对于非终结符$A$的**短语**

特别地，如果$A \Rightarrow \beta$，则称$\beta$是句型$\alpha \beta \delta$相对于非终结符$A$的**直接短语**

一个句型的**最左**直接短语称为该句型的**句柄**



**规范规约与规范推导**

定义：假定$\alpha$是文法$G$的一个句子，我们称序列
$$
\alpha_n,\ \alpha_{n-1}\ ,\cdots ,\alpha_0
$$
是$\alpha$的一个规范规约，如果这个序列满足：

* $\alpha_n = \alpha$
* $\alpha_0$ 为文法的开始符号，$\alpha_0 = S$
* 对任何$i,\ 0\le i \le n $，$\alpha_{i-1}$ $\alpha_i$ 经过把**句柄**替换成相应产生式左部符号而得到的



## 5.2 算符优先分析

先以四则运算为例进行分析

#### 四则运算的优先规则

先乘除后加减，同级从左到右



#### 算符优先分析的基本思想

* 起决定作用的是相邻的两个算符之间的优先关系
* 算符优先分析法就是定义算符之间的某种优先关系，借助于这种关系寻找“可规约串”并进行规约



#### 算符（终结符）优先关系

{% asset_img Snipaste_2022-02-14_17-03-25.png %}



#### 算符文法

定义：任意产生式的右部都不含有两个相继的**非终结符**的文法，即不含有如下形式的产生式右部：
$$
\cdots QR \cdots
$$
采用如下约定：

* $a$ 、$b$ 代表任意**终结符**
* $P$ 、$Q$、$R$ 代表任意非终结符
* $\cdots$ 代表由终结符和非终结符组成的任意序列，包括**空字**



#### 算符优先文法

假定$G$是一个不含$\varepsilon$产生式的**算符文法**，对于任何一对终结符$a$ 、$b$，我们说：

* $a \doteq b$ 当且仅当文法$G$中含有形如$P\rightarrow \cdots ab \cdots$ 或 $P\rightarrow \cdots aQb \cdots$的产生式；
* $a \lessdot b$ 当且仅当文法$G$中含有形如$P\rightarrow \cdots aR \cdots$ 的产生式，而$R \Rightarrow^+ b \cdots$ 或者 $R \Rightarrow^+ Qb \cdots$ ；
* $a \gtrdot b$ 当且仅当文法$G$中含有形如$P\rightarrow \cdots Rb \cdots$ 的产生式，而$R \Rightarrow^+ a \cdots$ 或者 $R \Rightarrow^+ aQ \cdots$ 。

如果一个算符文法$G$中的任何终结符对$(a,\ b)$至多只满足上述三种关系之一，那么文法$G$就是一个**算符优先文法**



#### 构造算符优先关系表

略



#### 素短语

素短语是这样一种短语，它至少含有一个终结符，并且，除它自身之外不含其它任何更小的素短语

**最左素短语**是指处于句型最左边的那个素短语



#### 寻找最左素短语

算符优先文法句型的一般形式写成：
$$
\#N_1a_1N_2a_2\cdots N_na_nN_{n+1} \#
$$
其中$a_i$是终结符，$N_i$是非终结符且可有可无



**定理**：一个算符优先文法$G$的任何句型的最左素短语是满足如下条件的**最左子串**$N_ja_j\cdots N_ia_iN_{i+1}$
$$
a_{j-1} \lessdot a_j \newline
a_j \doteq a_{j+1},\ \cdots\ ,\ a_{i-1} \doteq a_i\newline
a_i \gtrdot a_{i+1}
$$


## 5.3 LR分析法

L - left

R - reduce

从左到右进行规约的分析方法



### 5.3.1 LR分析器

**规范规约**的关键问题是寻找**句柄**

LR分析方法：把已经读取的和期望读取的字符抽象成状态；由栈顶的状态和现行的输入符号**唯一**确定每一步工作

{% asset_img Snipaste_2022-02-15_09-33-07.png %}



#### LR分析表

$ACTION[s,\ a]$ ：当状态 $s$面临输入符号$a$时应该采取的动作

$GOTO[s,\ X]$ ：状态$s$面临文法符号$X$时，下一个状态是什么

举例：文法$G(E)$:
$$
\begin{align}
E &\rightarrow E+T \newline
E &\rightarrow T \newline
T &\rightarrow T*F \newline
T &\rightarrow F \newline
F &\rightarrow (E) \newline
F &\rightarrow i
\end{align}
$$
其LR分析表为：

{% asset_img Snipaste_2022-02-15_09-53-08.png %}



#### LR文法

定义：对于一个文法，如果能够构造一张分析表，使得它的每个入口均是唯一确定的，则这个文法就称为LR文法



#### LR(k) 文法

定义：一个文法，如果能用一个每步顶多向前检查$k$个输入符号的LR分析器进行分析，则称这个文法为LR(k)文法



### 5.3.2 LR(0)项目集族和LR(0)分析表的构造

#### 活前缀

指规范句型的一个**前缀**，这中前缀**不含有**句柄**之后**的任何符号。

规范规约过程中，保证分析栈中总是**活前缀**，就说明分析采取的移进-规约动作是正确的



所以需要一种方法来**识别活前缀**，以确保规约过程的正确性

这个识别的工作可以使用**有限自动机（FA）**完成

两种构造识别活前缀的DFA的方法：

* 项目 $\rightarrow$ NFA $\rightarrow$ DFA
* Closure $\rightarrow$ GO $\rightarrow$ DFA



#### 项目 $\rightarrow$ NFA $\rightarrow$ DFA

**项目的定义**

文法$G$的每个产生式的右部添加一个圆点就成了$G$的一个项目

举例🌰
$$
\begin{align}
文法：&A \rightarrow XYZ\ 的四个项目\newline
&1.\ A \rightarrow \bullet XYZ \newline
&2.\ A \rightarrow X\bullet YZ \newline
&3.\ A \rightarrow XY\bullet Z \newline
&4.\ A \rightarrow  XYZ\bullet \newline
\end{align}
$$

* $A \rightarrow \alpha \bullet$ 称为归约项目
	* $S^{'} \rightarrow \alpha \bullet$ 称为接受项目
* $A \rightarrow \alpha \bullet a \beta$ 称为移进项目
* $A \rightarrow \alpha \bullet B \beta$ 称为待约项目



**由项目生成NFA的方法**

* 如果状态 $i$ 为 $X \rightarrow X_1 \cdots X_{i-1} \bullet X_{i} \cdots X_n$ ，状态 $j$ 为 $X \rightarrow X_1 \cdots X_{i-1}X_{i} \bullet X_{i+1} \cdots X_n$ ，那么就从状态$i$画一条标志为 $X_{i}$ 的有向边到状态 $j$ ；
* 如果状态 $i$ 为$X\rightarrow \alpha \bullet A \beta$，$A$为非终结符，那么就从状态 $i$ 画一条 $\varepsilon$ 边到所有状态 $A \rightarrow \bullet \gamma$



NFA $\rightarrow$ DFA ：使用子集法（看词法分析章节）



#### LR(0) 项目集规范族

构成识别一个文法活前缀的DFA的**项目集的全体**称为文法的LR(0)项目集规范族



#### 有效项目

如果存在如下规范推导
$$
S^{'} \Rightarrow^* \alpha A\omega \Rightarrow \alpha \beta_1 \beta_2 \omega
$$
我们就说项目$A \rightarrow \beta_1 \bullet \beta_2$ 对于活前缀 $\alpha \beta_1$是有效项目

**结论**：如果项目$A \rightarrow \alpha \bullet B\beta$ 对活前缀 $\eta = \delta \alpha$ 是有效的且 $B \rightarrow \gamma$ 是一个产生式，那么项目 $B \rightarrow \bullet \gamma$ 对 $\eta = \delta \alpha$ 也是有效的



#### Closure $\rightarrow$ GO $\rightarrow$ DFA



**闭包CLOSURE**

假定 $I$ 是文法 $G^{'}$ 的任一项目集，定义和构造 $I$ 的闭包 $CLOSURE(I)$ 如下：

* $I$ 中的任何项目都属于  $CLOSURE(I)$ 
* 如果 $A \rightarrow \alpha \bullet B \beta$ 属于 $CLOSURE(I)$ ，那么，对于任何关于 $B$ 的产生式 $B \rightarrow \gamma$，项目 $B\rightarrow \bullet \gamma$ 也属于 $CLOSURE(I)$ 



**GO函数**

为了识别活前缀，我们定义一个状态转换函数GO。 $I$ 是一个项目集， $X$ 是一个文法符号。函数值 $GO(I,\ X)$ 定义为
$$
GO(I,\ X)=CLOSURE(J)
$$
其中 $J=\{任何形如 A\rightarrow \alpha X \bullet \beta 的项目 \mid A\rightarrow \alpha \bullet X \beta 属于 I\}$



转换函数GO把项目集连接成一个DFA转换图



本质上讲，

* 项目 $\rightarrow$ NFA $\rightarrow$ DFA
* Closure $\rightarrow$ GO $\rightarrow$ DFA

两种构造识别活前缀DFA的方法是相同的，只是表达方式不同。

一个文法和它对应的识别活前缀的DFA：

{% asset_img Snipaste_2022-02-15_20-21-28.png %}



#### LR(0)分析表的构造

* 令每个项目集$I_k$的下标$k$作为分析器的状态，包含项目$S^{'} \rightarrow \bullet S$ 的集合 $I_k$ 的下标 $k$ 为分析器的**初态**
* 如果项目$A\rightarrow \alpha \bullet a \beta$属于$I_k$且$GO(I_k,a)=I_j$，$a$为终结符，则置 $ACTION[k,\ a]$为 $sj$
* 如果项目$A\rightarrow \alpha \bullet $属于$I_k$，那么对于任何终结符 $a$ ，置 $ACTION[k,\ a]$ 为 $rj$ （$j$ 是产生式 $A\rightarrow \alpha \bullet$ 在文法中的序号）
* 如果项目 $S^{'} \rightarrow S \bullet$ 属于$I_k$ ，则置 $ACTION[k,\ \#]$ 为 $acc$
* 如果$GO(I_k,\ A) = I_j$ ， $A$ 为非终结符，则置$GOTO[k,\ A]=j$
* 不能用上述规则完善表格信息的空白格都填上报错标志



### 5.3.3 SLR分析表的构造

使用更精细的方法解决LR(0)分析表中的一些冲突



### 5.3.4 规范LR分析表的构造

解决SLR分析表存在的问题



**YACC** ：语法分析器产生器



# 第六章 属性文法和语法制导翻译

## 6.1 属性文法

在上下文无关文法的基础上，为每个文法符号配备若干相关的值，称为属性。

* 属性代表与文法符号相关信息，如类型、值、代码序列、符号表内容等等
* 属性可以畸形计算和传递
* 语义规则：对于文法的每一个产生式配备一组属性的计算规则



**属性**

* 综合属性：自下而上传递信息
	* 在语法树中，一个节点的综合属性的值由其**子节点**和它**本身**的属性确定
	* 使用自底向上的方法在每一个节点处使用语义规则计算综合属性的值
	* 只使用综合属性的属性文法称为**S-属性文法**

* 继承属性：自上而下传递信息
	* 在语法树中，一个节点的继承属性由其父节点、兄弟节点和本身的属性确定
	* 用继承属性来表示程序设计语言结构中的上下文依赖关系很方便



## 6.2 基于属性文法的处理方法

编译部分过程：$输入串 \rightarrow 语法树 \rightarrow 按照语义规则计算属性$



由源程序的**语法结构**所**驱动**的处理办法就是**语法制导翻译**

对输入符号串的**翻译**也就是根据**语义规则**进行计算的结果



**基于属性文法的处理方法**：

* 依赖图
* 树遍历
* 一遍扫描



### 6.2.1 依赖图

一棵语法树中的节点的继承属性和综合属性之间的相互**依赖关系**可以用依赖图（有向图）来描述



依赖图的创建：

* 为每一个包含过程（函数）调用的语义规则引入一个虚综合属性 $b$，这样把每一个语义规则都写成

$$
b:=f(c_1,c_2,\cdots,c_k)
$$

* 依赖图中为每一个**属性**设置一个节点，如果属性 $b$ 依赖于属性 $c$ ，则从属性 $c$ 的节点有一条有向边连到属性 $b$ 的节点

举例🌰：

{% asset_img Snipaste_2022-02-16_09-56-24.png %}

图中的 $addtype$ 属性就是引入的虚综合属性，它依赖于 $entry$ 和 $in$ 属性



### 6.2.2 树遍历

通过树遍历的方法计算属性的值：

* 假设语法树已经建立，且树中已带有开始符号的继承属性和终结符的综合属性
* 以某种次序遍历语法树，直到计算出所有属性
	* 深度优先，从左到右遍历

树遍历伪代码

```
while 还有未计算的属性 do
	VisitNode(S) // S 是开始符号

procedure VisitNode(N:Node);
begin
	if N是一个非终结符 then
		//假设它的一个产生式为 N -> X1X2...Xm
		for i:=1 to m do
			if Xi是非终结符 then
			begin
				计算Xi的所有能够计算的继承属性;
				VisitNode(Xi);
			end;
	计算N的所有能够计算的综合属性;
end
```



依赖图和树遍历的翻译方法本质上相同。树遍历方法用代码实现更加方便。



### 6.2.3 一遍扫描

一遍扫描的处理方法是在语法分析的**同时**计算属性值



**语法制导翻译方法**

直观上说就是为文法中每个产生式配上一组语义规则，并且在语法分析的同时执行这些语义规则



#### 抽象语法树（Abstract Syntax Tree）

在语法树中去掉那些对翻译不必要的信息，从而获得更有效的源程序中间表示。这种经过变换之后的语法树称之为抽象语法树。

**建立抽象语法树**

* `mknode(op, left, right)` 建立一个运算符号节点，标号是 `op`，两个域 `left` 和 `right` 分别指向左子树和右子树
* `mkleaf(id, entry)` 建立一个标识符节点，标号为 `id` ，`entry` 指向标识符在符号表中的入口
* `mkleaf(num, val)` 建立一个数节点，标号为 `num` ，`val` 存放数值



## 6.3 S-属性文法的自下而上计算

S-属性文法是只含有**综合属性**的文法，适合自下而上的语法分析方法

综合属性可以在分析输入符号串的同时由自下而上的分析器来计算

分析器可以保存与栈中文法符号有关的综合属性值，每当进行归约时，新的属性值就由栈中正在进行归约的产生式右边符号的属性值来计算



## 6.4 L-属性文法和自顶向下翻译

#### L-属性文法

如果对于一个属性文法的每个产生式 $A \rightarrow X_1X_2\cdots X_n$ ，其语义规则中的每个属性是**综合属性**或者是 $X_j(1 \le j \le n)$ 的一个**继承属性**且这个继承属性仅仅依赖于：

(1)产生式中$X_j$左边符号$X_1,\ X_2,\ \cdots \ , \ X_{j-1}$的属性

(2)$A$的继承属性

那么这个属性文法就是L-属性文法

### 6.4.1 翻译模式

语义规则只给出了属性的计算方法，但是没有给出属性的计算**顺序**等信息

在**翻译**模式中，和文法符号相关的属性和语义规则，用花括号 $\{\}$ 括起来，插入到产生式右部的合适位置上



**设计翻译模式的原则**

* 必须保证当某个动作引用一个属性时，该属性是有定义的
* L-属性文法本身就能确保每个动作不会引用尚未计算出来的属性



**建立翻译模式**

* 当只需要综合属性时：为每一个语义规则建立一个包含赋值的动作，并把这个动作放在相应的产生式右边的末尾
* 如果既有综合属性又有继承属性，那么必须保证：
	* 产生式右边的符号的继承属性必须在这个符号以前的动作中计算出来
	* 一个动作不能引用这个动作右边的符号的综合属性
	* 产生式左边非终结符的综合属性只有在它所引用的所有属性都计算出来以后才能计算

### 6.4.2 自顶向下翻译

在 4.2 节中指出了自顶向下的语法分析要解决文法左递归的问题。自顶向下翻译要在消除左递归的同时考虑属性的变化。

自顶向下翻译就是要在进行文法等价变换的同时做语义规则的等价变换。

### 6.4.3 递归下降翻译器的设计

设计方法：

* 定义参数和变量
	* 对每个非终结符$A$构造一个函数过程，对$A$的每个继承属性设置一个形式参数
	* 函数的返回值为$A$的综合属性
	* $A$对应的函数过程中，为出现在$A$的产生式中的每一个文法符号的每一个属性都设置一个局部变量

* 函数的功能
	* 非终结符$A$对应的函数过程中，根据当前的输入符号决定使用哪个产生式候选

* 每个产生式对应的程序代码
	* 按照从左到右的次序，对于终结符、非终结符和语义动作分别做以下工作
		* 对于带有综合属性$x$的终结符$X$，把$x$的值存入为$X.x$设置的变量中。然后产生一个匹配$X$的调用，并继续读入一个输入符号。
		* 对于每个非终结符$B$，产生一个右边带有函数调用的赋值语句$c=B(b_1,\ b_2,\ \cdots\ ,\ b_k)$，其中，$b_1,b_2,\cdots ,b_k$是为$B$的继承属性设置的变量，$c$是为$B$的综合属性设置的变量。
		* 对于语义动作，把动作的代码抄进分析器中，用代表属性的变量来代替对属性的每一次引用



# 第七章 语义分析和中间代码产生

## 7.1 中间语言

中间语言是独立于机器平台的，它的复杂性介于源语言和目标语言之间。

常用的中间语言有：

* 后缀式，逆波兰表示
* 图表示：DAG、抽象语法树
* 三地址代码

### 7.1.1 后缀式

一个表达式$E$的后缀形式可以如下定义：

* 如果$E$是一个变量或者常量，则$E$的后缀式是$E$自身
* 如果$E$是$E_1\ op\ E_2$形式的表达式，其中 $op$是任何二元操作符，那么$E$的后缀式为$E_1^{'}E_2^{'}\ op$，其中$E_1^{'}$、 $E_2^{'}$ 分别是 $E_1$、$E_2$的后缀式
* 如果$E$是$(E_1)$形式的表达式，则$E_1$的后缀式就是$E$的后缀式

### 7.1.2 图表示法

#### DAG 无循环有向图

#### 抽象语法树

### 7.1.3 三地址代码

`x := y op z`

三地址代码可以看成是抽象语法树或DAG的一种线性表示

三地址语句的种类：

{% asset_img Snipaste_2022-02-17_11-25-55.png %}

#### 间接三元式

能够统一表示三地址代码。



## 7.3 赋值语句的翻译

### 7.3.1 简单的算术表达式及赋值语句

#### `id:=E` 的翻译

* 对表达式$E$求值并置于变量$T$中
* `id.place:=T` 将 `id.place` 指向计算出来的 `T`



## 使用S-属性文法生成赋值语句的三地址代码

先来规定一些符号的意义：

* 非终结符号$S$有综合属性 `S.code` ，它代表赋值语句$S$的三地址代码
* 非终结符号$E$有以下两个属性：
	* `E.place`存放$E$的值
	* `E.code`存放对$E$求值的三地址语句序列
	* 函数`newtemp`返回临时变量，且确保所有由它产生的临时变量不重名



### 7.3.2 数组元素的引用

数组元素引用是类似于这样的产生式：
$$
X:=A[i_1,\ i_2,\ \cdots \ ,\ i_k]+Y\newline
A[i_1,\ i_2,\ \cdots \ ,\ i_k]:=X+Y
$$

#### 数组元素地址的计算

设 $A$ 为 $n$ 维数组，按行存放，每个元素宽度为 $w$

* $low_i$为第$i$维的下界
* $up_i$为第$i$维的上界
* $n_i$为第$i$维可取值的个数（$n_i = up_i - low_i + 1$）
* $base$为$A$的第一个元素的地址

那么元素$A[i_1,\ i_2,\ \cdots \ ,\ i_k]$的地址计算公式为
$$
((((\cdots i_1n_2+i_2)n_3+i_3)\cdots)n_k+i_k)\times w + \newline base-((\cdots((low_1n_2+low_2)n_3+low_3)
\cdots)n_k+low_k)\times w
$$
引入下列语义变量或者语义过程

* `Elist.ndim` 下标个数计数器
* `Elist.place` 临时变量，存放已经形成的`Elist`中的下标表达式计算出来的值
* `Elist.array` 数组名
* `limit(array, j)` 返回数组`array`的第`j`维的长度

每个代表变量的非终结符$L$有两项语义值

* `L.place`
	* 如果$L$是简单变量$i$，`L.place`指$i$的符号表入口
	* 如果$L$是下标变量，`L.place`指存放不变部分的临时变量的整数码

* `L.offset`
	* 如果$L$是简单变量$i$，`L.offset` 是 `null`
	* 如果$L$是下标变量，`L.offset` 是存放可变部分的临时变量的整数码

带数组下标引用的文法
$$
\begin{align}
S&\rightarrow L:=E\newline
E&\rightarrow E+E\newline
E&\rightarrow(E)\newline
E&\rightarrow L\newline
L&\rightarrow Elist] \newline
L&\rightarrow id \newline
Elist&\rightarrow Elist,E \newline
Elist& \rightarrow id[E
\end{align}
$$
该文法的翻译模式：

**$S\rightarrow L := E$**

```shell
{
	if L.offset=null then
		emit(L.place ':=' E.place)
	else
		emit(L.place'['L.offset']'':=' E.place)
}
```

**$E\rightarrow E_1 + E_2$**

```shell
{
	E.place := newtemp
	emit(E.place ':=' E_1.place '+' E_2.place)
}
```

**$E\rightarrow (E_1)$**

```shell
{ E.place:=E_1.place }
```

**$E\rightarrow L$**

```shell
{
	if L.offset=null then
		E.place:=L.place
	else begin
		E.place:=newtemp
		emit(E.place ':=' L.place'['L.offset']')
	end
}
```

**$Elist \rightarrow id[E$**

```shell
{
	Elist.place:=E.place
	Elist.ndim:=1 # 刚刚开始处理中括号之内的表达式，只处理了数组索引的第一维
	Elist.array:=id.place
}
```

**$Elist\rightarrow Elist_1,E $**

```shell
{
	t:=newtemp
	m:=Elist_1.ndim+1
	emit(t ':=' Elist_1.place '*' limit(Elist_1.array, m))
	emit(t ':=' t '+' E.place)
	Elist.place:=t
	Elist.ndim:=m
	Elist.array:=Elist_1.array
}
```

**$L\rightarrow Elist]$**

```shell
{
	L.place:=newtemp
	emit(L.place ':=' Elist.array '-' C)
	
	L.offset:=newtemp
	emit(L.offset ':=' w '*' Elist.place)
}
```

**$L\rightarrow id$**

```shell
{ 
	L.place:=id.place
	L.offset:=null
}
```



#### 类型转换赋值表达式的翻译

给非终结符增加一个$type$属性，比如，用`E.type`表示非终结符$E$的类型属性

产生式$E\rightarrow E_1\ op\ E_2$的语义规则中关于类型的部分可以定义为：

```shell
{
	if E_1.type=integer and E_2.type=integer
		E.type:=integer
	else
		E.type:=real
}
```



产生式 $E\rightarrow E_1 + E_2$ 的语义动作

```shell
{ 
	E.place:=newtemp;
    if E1.type=integer and E2.type=integer then
    begin
        emit (E.place ':=' E 1.place 'int+' E 2.place);
        E.type:=integer
    end
    else if E1.type=real and E2.type=real then begin
    	emit (E.place ':=' E 1.place 'real+' E 2.place);
    	E.type:=real
    end
    else if E1.type=integer and E2.type=real then begin
    	u:=newtemp;
    	emit (u ':=' 'inttoreal' E 1.place);
        emit (E.place ':=' u 'real+' E 2.palce);
        E.type:=real
    end
    else if E1.type=real and E2.type=integer then begin
        u:=newtemp;
        emit (u ':=' 'inttoreal' E 2.place);
        emit (E.place ':=' E 1.place 'real+' u);
        E.type:=real
    end
    else E.type:=type_error
}
```



## 7.4 布尔表达式的翻译

产生布尔表达式的文法
$$
E\rightarrow E\ or\ E\ |\ E\ and\ E\ |\ not\ E\ |\ (E)\ |\ i\ rop\ i\ |\ i
$$
$rop$ 指关系运算符（<、 >、 <=……）

### 7.4.1 数值表示方法

$a\ or\ b\ and\ not\ c$ 翻译成
$$
\begin{align}
T_1&:=not\ c\newline
T_2&:=b\ and\ T_1\newline
T_3&:=a\ or\ T_2
\end{align}
$$
$a<b$翻译成
$$
\begin{align}
100:&\ if\ a<b\ then\ 1\ else\ 0\newline
101:&\ T:=0\newline
102:&\ goto\ 104\newline
103:&\ T:=1\newline
104:&\newline
\end{align}
$$


### 7.4.2 作为条件控制的布尔翻译

先定义一些规则和符号：

* `newlabel` 函数每次返回一个新的符号标号
* 对于每个布尔表达式$E$，引用两个标号
	* `E.true`是$E$为真时控制流转向的标号
	* `E.false`是$E$为假时控制流转向的标号

语义规则：

**$E\rightarrow E_1\ or\ E_2$**

{% asset_img Snipaste_2022-02-17_19-52-29.png %}

**$E\rightarrow E_1\ and\ E_2$**

{% asset_img Snipaste_2022-02-17_19-53-44.png %}

**$E\rightarrow not\ E_1$** 和 **$E\rightarrow (E_1) $**

{% asset_img Snipaste_2022-02-17_19-55-03.png %}

**$E\rightarrow id_1\ relop\ id_2$**

{% asset_img Snipaste_2022-02-17_19-56-06.png %}



#### 一遍扫描实现布尔表达式的翻译

* 采用四元式形式
* 把四元式存入一个数组之中，数组下标就代表四元式的标号
* 约定：
	* `(jnz, a, -, p)` : `if a goto p`
	* `(jrop, x, y, p) ` ：`if x rop y goto p`
	* `(j, -, -, p) ` ：`goto p`

一遍扫描的思路：为每个布尔表达式添加 `truelist` 和 `falselist` 两个属性，它们记录表达式在未来某个时刻需要补充的地址。在扫描到关键时刻时将这些地址**回填**就能使用一次扫描翻译布尔表达式



## 7.5 控制语句的翻译

$$
\begin{align}
S\rightarrow&\ if\ E\ then\ S_1\newline
&|\ if\ E\ then\ S_1\ else\ S_2\newline
&|\ while\ E\ do\ S_1
\end{align}
$$

$if-then$语句

{% asset_img Snipaste_2022-02-17_21-00-06.png %}

语义规则
$$
\begin{align}
E.true&:=newlabel\newline
E.false&:=S.next\newline
S_1.next&:=S.next\newline
S.code&:=E.code\ ||\ gen(E.true\ ':')\ ||\ S_1.code\newline
\end{align}
$$


$if-then-else$语句

{% asset_img Snipaste_2022-02-17_21-05-08.png %}

语义规则
$$
\begin{align}
E.true&:=newlabel\newline
E.false&:=newlabel\newline
S_1.next&:=S.next\newline
S_2.next&:=S.next\newline
S.code:=E.code\ &||\ gen(E.true ':')\ ||\ S_1.code\newline ||\ gen&('goto'\ S.next)\ ||\ gen(E.false ':')\ ||\ S_2.code
\end{align}
$$


$while$语句

{% asset_img Snipaste_2022-02-17_21-11-23.png %}

语义规则
$$
\begin{align}
S.begin&:=newlabel\newline
E.true&:=newlabel\newline
E.false&:=S.next\newline
S_1.next&:=S.begin\newline
S.code:=gen&(S.begin ':')||E.code\newline
&||gen(E.true ':')||S_1.code||gen('goto'S.begin)
\end{align}
$$


控制语句的翻译同样可以使用**一遍扫描**实现



# 第八章 符号表

## 8.1 符号表的组织与作用

| 名字 | 信息 |
| :--: | :--: |
|      |      |

按名字的不同种属建立多张符号表，如常数表、变量表、过程名表……



符号表的作用：

* 一致性检查和作用域分析
* 辅助代码生成



## 8.2 整理和查找

### 线性查找

{% asset_img Snipaste_2022-02-18_08-50-07.png %}

### 二分查找

{% asset_img Snipaste_2022-02-18_08-51-37.png %}

### 杂凑查找

{% asset_img Snipaste_2022-02-18_08-53-32.png %}



## 8.3 名字的作用范围

作用域：一个名字能被使用的区域范围

#### 名字作用域分析

两种做法：

* 引入“过程编号”属性。查找时，先查找本过程编号的名字，查不到再查找外层过程编号的名字，如果查到最外层过程仍然没有找到，则报错
* 按照“栈”式思想组织符号表。查找时，从后往前查找，碰到的第一个名字就是想要查找的



## 8.4 符号表的内容

符号表的信息栏中登记了每个名字的有关性质

* 类型：整型、实型、布尔型……
* 种属：简单变量、数组、过程……
* 大小：长度
* 相对数：分配给该名字的存储单元的相对地址



# 第九章 运行时存储空间组织

## 9.1 目标程序运行时的活动

以Pascal为例，假定程序由若干个过程组成：

* 一个过程的活动指的是该过程的一次执行
* 过程P一个活动的生存期，指的是从执行该过程第一步操作到最后一步操作之间的操作序，包括执行P时调用其它过程花费的时间
* 过程可以是**递归**的



#### 参数传递

过程定义：

```pascal
procedure add(x, y: integer; var z: integer)
begin
	z:=x+y;
end;
```

过程调用：

```pascal
add(a, b, c);
```

**参数传递方式**

* 传地址
	* 把实参的地址传给形参
	* 传递的方法：
		* 调用段先把实在参数的地址传到被调用段可以拿到的地方
		* 程序控制转入被调用段之后，被调用段先把实在参数的地址抄到自己相应的形式单元中
		* 过程体对形式参数的引用域赋值被处理成对形式单元的间接访问  

* 得结果
* 传值
	* 把实参的值传给形参
* 传名
	* 过程调用的作用相当于把被调用段的过程体抄到调用出现的地方，但把其中任一出现的形式参数替换成相应的实参



## 9.2 运行时存储器的划分

一个目标程序运行所需的存储空间：

* 存放**目标代码**的空间
* 存放**数据项目**的空间
* 存放程序运行的**控制或连接数据**所需单元

{% asset_img Snipaste_2022-02-18_12-51-41.png %}

### 9.2.2 活动记录

假定某种编程语言有这样的特点：

* 允许过程递归调用、允许过程含有可变数组
* 过程（函数）定义不允许嵌套，比如C

这样的变成语言通常采用**栈式存储分配**

#### 活动记录

* 运行时，每当进入一个过程就在栈顶推入一个相应的活动记录
* 这条记录里含有连接数据、形式单元、局部变量、局部数组的内情向量和临时工作单元等等

### 9.2.3 存储分配策略

* 静态分配
	* 如果在编译的时候就能确定数据空间的大小就可以使用静态分配
* 动态分配
	* 允许递归过程和动态申请释放内存等特性，编译时无法确定运行时数据空间的大小，就必须采用动态分配策略。方法有：
		* 栈式动态分配
		* 堆式动态分配



## 9.3 静态存储管理

典型语言：FORTRAN

**FORTRAN**程序的特点

* 整个程序所需要数据空间的总量在编译时完全确定
* 每个数据名的地址可以静态地分配



按**数据区**组织存储

* 每个程序段定义一个**局部数据区**，用来存放程序段中未出现在COMMON里的局部名的值
* 每个公用块定义一个**公用数据区**，用来存放公用块里各个名字的值
* 每个数据区有一个编号，地址分配时，在符号表中，对每个数据名登记其所属数据区编号及在该区中的相对位置
* 每个**局部数据区**的内容及存放次序如下

{% asset_img Snipaste_2022-02-18_13-31-53.png %}

其中形式单元指的是对形参传参方式的处理

## 9.5 嵌套过程语言的栈式实现

典型语言：PASCAL

**PASCAL**语言的特点：不仅允许过程的递归调用，而且允许过程**定义的嵌套**

### 9.5.1 非局部名字的访问的实现

方法一：静态链和活动记录

**静态链**：指向本过程的**直接**外层过程的活动记录的起始地址。（谁**定义**了我我就指向谁）

动态链：指向调用本过程的过程的活动记录起始地址。（谁**调用**了我我就指向谁）



方法二：嵌套层次显示表display

用display表记录嵌套调用的层次



# 第十章 优化

## 10.1 概述

优化：对程序进行**等价**变换，使得从变换后的程序出发，能生成更有效的目标代码

优化的不同**级别**：

* 局部优化
* 循环优化
* 全局优化



## 10.2 局部优化 主要借助DAG（无环有向图）

引入**基本块**的概念：程序中顺序执行的语句序列，其中只有一个入口和一个出口。入口就是其中第一个语句，出口就是其中最后一个语句。

有三地址语句 $x:=y+z$ ，称对$x$**定值**并**引用**$y$和$z$

基本块中的一个名字在程序中的某个给定点是**活跃**的，是指如果在程序中它的值在该点以后被引用。

局限于基本块范围内的优化称为局部优化。



#### 划分四元式程序为基本块

**1.找到入口语句**

* 程序的第一个语句
* 能由条件转移语句或无条件转移语句转移到的语句
* 紧跟在条件转移语句后面的语句

**2.确定入口语句所属的基本块**

* 一条入口语句到下一条入口语句（不包含下一条入口语句）之间的语句序列
* 一条入口语句到某一转移语句（包括该转移语句）之间的语句序列
* 一条入口语句到某一**停语句**（包括该停语句）之间的语句序列

**3.未在基本块内的语句可以删除**



#### 流图

* 以基本块为节点
* 入口语句为程序第一条语句的基本块是流图的**首节点**
* 如果在某个执行顺序中，基本块 $B_2$ 紧接在基本块 $B_1$ 之后**执行**，则从 $B_1$ 到 $B_2$ 有一条**有向边**  



#### 局部优化措施

* 合并已知量

* 临时变量改名
* 交换语句的位置
* 代数变换



### 10.2.2 基本块的DAG表示及其应用

DAG: 无环路有向图

{% asset_img Snipaste_2022-02-18_20-26-47.png %}

根据四元式构造DAG：

**0型**

{% asset_img Snipaste_2022-02-18_20-28-37.png %}

**1型**

{% asset_img Snipaste_2022-02-18_20-29-13.png %}

**2型**

{% asset_img Snipaste_2022-02-18_20-29-49.png %}

**2型**

{% asset_img Snipaste_2022-02-18_20-30-41.png %}

**3型**

{% asset_img Snipaste_2022-02-18_20-31-22.png %}

**0型：goto(s)**

{% asset_img Snipaste_2022-02-18_20-32-43.png %}



规定：函数`Node(A)`返回标识符$A$所在的节点，如果不存在这样的节点，则返回 `null`

对基本块中的每个四元式，执行一下步骤构造对应的DAG：

1.准备操作数的节点

{% asset_img Snipaste_2022-02-18_20-44-35.png %}

2.合并已知量

{% asset_img Snipaste_2022-02-18_20-46-01.png %}

3.删除公共子表达式

{% asset_img Snipaste_2022-02-18_20-46-47.png %}

4.删除无用赋值

{% asset_img Snipaste_2022-02-18_20-47-13.png %}



## 10.3 循环优化

主要措施：

* 代码外提
* 强度削弱（乘法变加法等等）
* 变换循环控制条件
* 循环展开
* 循环合并



# 第十一章 代码生成

代码生成的任务：把中间代码变换成目标代码

**目标代码的三种形式**：

* 绝对指令代码：所有地址都已经定位，能够立即执行
* 可重定位指令代码：地址没有确定，经过链接重定位后才能运行
* 汇编指令代码（课上用的目标代码类型）
