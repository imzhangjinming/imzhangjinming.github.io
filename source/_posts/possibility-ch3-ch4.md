---
title: 概率论与数理统计教程 茆诗松 第三、四章
date: 2022-04-10 18:30:42
categories:
- aeronautics
tags:
---

补习基础数理知识之概率论

<!--more-->

# 第三章 多维随机变量及其分布

## 3.1 多维随机变量及其联合分布

### 3.1.1 多维随机变量

**定义** 如果$X_1(\omega),\ X_2(\omega),\cdots,\ X_n(\omega)$是定义在同一个样本空间$\Omega=\{\omega\}$上的$n$个随机变量，则称
$$
X(\omega)=(X_1(\omega),\ X_2(\omega),\cdots,\ X_n(\omega))
$$
为$n$维随机变量。

**这个定义的关键点在于：所有随机变量定义在同一个样本空间上**

### 3.1.2 联合分布函数

**定义** 对任意$n$个实数$x_1,x_2,\cdots,x_n$，则$n$个事件$\{X_1\le x_1\},\{X_2\le x_2\},\cdots,\{X_n\le x_n\}\}$同时发生的概率
$$
F(x_1,x_2,\cdots,x_n)=P(X_1\le x_1,X_2\le x_2,\cdots,X_n\le x_n)
$$
称为$n$维随机变量$(X_1,X_2,\cdots,X_n)$的联合分布函数。

### 3.1.4 联合密度函数

**定义** 如果存在二元非负函数$p(x,y)$，使得二维随机变量$(X,Y)$的分布函数$F(X,Y)$可以表示为
$$
F(x,y)=\int_{-\infty}^{x}\int_{-\infty}^{y}p(u,v)dvdu\ ,
$$
则称$(X,Y)$为二维连续随机变量，称$p(u,v)$为$(X,Y)$的联合密度函数。

### 3.1.5 常用多维分布

#### 多项分布 ：二项分布的推广

进行$n$次独立重复试验，如果每次试验有$r$个互不相容结果：$A_1,A_2,\cdots\,A_r$之一发生，且每次试验中$A_i$发生的概率为$p_i=P(A_i),i=1,2,\cdots,r$，且$p_1+p_2+\cdots+p_n=1$.记$X_i$为$n$次独立重复试验中$A_i$出现的次数，$i=1,2,\cdots,r$.则$(X_1,X_2,\cdots,X_r)$取值$(n_1,n_2,\cdots,n_r)$的概率，即$A_1$出现$n_1$次，$A_2$出现$n_2$次，……，$A_r$出现$n_r$次的概率为
$$
\begin{aligned}
P(X_1=n_1,X_2=n_2,\cdots,X_r=n_r)=\frac{n!}{n_1!n_2!\cdots n_r!}p_1^{n_1}p_2^{n_2}\cdots p_r^{n_r},
\end{aligned}
$$
其中$n=n_1+n_2+\cdots n_r$

这个分布叫做$r$项分布，$r=2$是就是二项分布。



#### 多维超几何分布 不放回取球模型

$$
P(X_1=n_1,X_2=n_2,\cdots,X_r=n_r)=\frac{\displaystyleC_{N_1}^{n_1}C_{N_2}^{n_2}\cdots C_{N_r}^{n_r}}{\displaystyleC_{N}^{n}}
$$

其中$n=n_1+n_2+\cdots n_r$



#### 多维均匀分布 $(X_1,X_2,\cdots,X_n)\sim U(D)$

设$D$为$R^n$中的一个有界区域，其度量（平面的为面积，空间的为体积等）为$S_D$，如果多维随机变量$(X_1,X_2,\cdots,X_n)$的联合密度函数为
$$
p(x_1,x_2,\cdots ,x_n)=
\begin{cases}
\begin{alignat}{2}
\frac{1}{S_D}& & &,\ (x_1,x_2,\cdots,x_n)\in D,\\
0& & &,\ 其他.
\end{alignat}
\end{cases}
$$
则称$(X_1,X_2,\cdots,X_n)$服从$D$上的多维均匀分布



#### 二元正态分布 $(X,Y)\sim N(\mu_1,\mu_2,\sigma_1^2,\sigma_2^2,\rho)$

$$
\begin{multline}
p(x,y)=\frac{1}{2\pi\sigma_1\sigma_2\sqrt{1-\rho^2}}\exp\{-\frac{1}{2(1-\rho^2)}[\frac{(x-\mu_1)^2}{\sigma_1^2}\\
-2\rho\frac{(x-\mu_1)(y-\mu_2)}{\sigma_1\sigma_2}+\frac{(y-\mu_2)^2}{\sigma_2^2}]\},-\infty<x,y<\infty,
\end{multline}
$$

其中
$$
-\infty<\mu_1,\mu_2 <\infty,\quad \sigma_1,\sigma_2>0,\quad -1\le\rho\le1
$$
$\rho$是$X$和$Y$的相关系数



### 习题3.1

**1.100件产品中有50件一等品、30件二等品、20件三等品。从中任取5件，以$X,Y$分别表示取出的5件中一等品、二等品的件数，在以下情况下求$(X,Y)$的联合分布列。（1）不放回抽取；（2）有放回抽取。**

不放回取球符合多维超几何分布
$$
P(X=x,Y=y)=\frac{\displaystyleC_{50}^{x}C_{30}^{y}C_{20}^{5-x-y}}{\displaystyleC_{100}^{5}}
$$

| x\y  |     0     |    1     |    2     |    3    |    4     |    5     |
| :--: | :-------: | :------: | :------: | :-----: | :------: | :------: |
|  0   | 0.0002059 | 0.001931 | 0.006587 | 0.01025 | 0.007280 | 0.001893 |
|  1   | 0.003218  | 0.02271  | 0.05489  | 0.05393 | 0.01820  |    0     |
|  2   |  0.01855  | 0.09274  |  0.1416  | 0.06606 |    0     |    0     |
|  3   |  0.04946  |  0.1562  |  0.1132  |    0    |    0     |    0     |
|  4   |  0.06618  | 0.09177  |    0     |    0    |    0     |    0     |
|  5   |  0.02814  |    0     |    0     |    0    |    0     |    0     |

有放回抽取符合多项分布
$$
P(X=x,Y=y)=\frac{5!}{x!y!(5-x-y)!}0.5^x0.3^y0.2^{5-x-y}
$$

| x\y  |    0    |    1    |   2    |   3    |    4    |    5    |
| :--: | :-----: | :-----: | :----: | :----: | :-----: | :-----: |
|  0   | 0.00032 | 0.0024  | 0.0072 | 0.0108 | 0.0081  | 0.00243 |
|  1   |  0.004  |  0.024  | 0.054  | 0.0540 | 0.02025 |    0    |
|  2   |  0.02   |  0.09   | 0.1350 | 0.0675 |    0    |    0    |
|  3   |  0.05   |  0.15   | 0.1125 |   0    |    0    |    0    |
|  4   | 0.0625  | 0.09375 |   0    |   0    |    0    |    0    |
|  5   | 0.03125 |    0    |   0    |   0    |    0    |    0    |

**7.设二维随机变量$(X,Y)$的联合密度函数为**
$$
p(x,y)=
\begin{cases}
\begin{alignat}{2}
4xy,& &\quad &0<x<1,0<y<1,\\
0,& & &其他.
\end{alignat}
\end{cases}
$$
**求：**

**(1) $P(0<X<0.5,0.25<Y<1)$;
(2) $P(X=Y)$;
(3) $P(X<Y)$;
(4) $(X, Y)$ 的联合分布函数.**

(1)
$$
P(0<X<0.5,0.25<Y<1)=4\int_{0}^{0.5}\int_{0}^{1}xy \mathrm{d}y \mathrm{d}x =\frac{15}{64}
$$
(2) 给的积分区域是一条直线，结果为 0

(3)
$$
P(X<Y)=4\int_{0}^{1}\int_{0}^{y}xy \mathrm{d}x \mathrm{d}y 
=4\int_{0}^{1}\frac{1}{2}y^3\mathrm{d}y=\frac{1}{2}
$$
(4)
$$
F(x,y)=
\begin{cases}
\begin{alignat}{2}
0,& & \quad & x<0,y<0,\\
x^2y^2,& & & 0<x<1,0<y<1,\\
y^2,& & & x>1,0<y<1,\\
x^2,& & & 0<x<1,y>1,\\
1,& & & x>1,y>1.\\

\end{alignat}
\end{cases}
$$

## 3.2 边际分布与随机变量的独立性

### 3.2.1 边际分布函数

如果在二维随机变量$(X,Y)$的联合分布函数$F(x,y)$中令$y\to \infty$，由于${Y<\infty}$为必然事件，故可得
$$
\lim _{y \rightarrow \infty} F(x, y)=P(X \leqslant x, Y<\infty)=P(X \leqslant x)
$$
这是由$(X,Y)$的联合分布函数$F(x,y)$求得的$X$的分布函数，被称为$X$的边际分布，记为$F_X(x)=F(x,\infty)$

### 3.2.2 边际分布列

在二维离散随机变量 $(X, Y)$ 的联合分布列 $\left\{ P\left(X=x_{i}, Y=y_{j}\right)\right\}$ 中, 对 $j$ 求和所 得的分布列
$$
\sum_{j=1}^{\infty} P\left(X=x_{i}, Y=y_{j}\right)=P\left(X=x_{i}\right), \quad i=1,2, \cdots
$$
被称为 $X$ 的边际分布列. 类似地, 对 $i$ 求和所得的分布列
$$
\sum_{i=1}^{\infty} P\left(X=x_{i}, Y=y_{j}\right)=P\left(Y=y_{j}\right), \quad j=1,2, \cdots
$$
被称为 $Y$ 的边际分布列.

### 3.2.3 边际密度函数

$$
\begin{aligned}
&p_{X}(x)=\int_{-\infty}^{\infty} p(x, y) \mathrm{d} y, \\
&p_{Y}(y)=\int_{-\infty}^{\infty} p(x, y) \mathrm{d} x .
\end{aligned}
$$

### 3.2.4 随机变量间的独立性

**定义** 设 $n$ 维随机变量 $\left(X_{1}, X_{2}, \cdots, X_{n}\right)$ 的联合分布函数为 $F\left(x_{1}, x_{2}\right.$, $\left.\cdots, x_{n}\right), F_{i}\left(x_{i}\right)$ 为 $X_{i}$ 的边际分布函数. 如果对任意 $n$ 个实数 $x_{1}, x_{2}, \cdots, x_{n}$, 有
$$
F\left(x_{1}, x_{2}, \cdots, x_{n}\right)=\prod_{i=1}^{n} F_{i}\left(x_{i}\right),
$$
则称 $X_{1}, X_{2}, \cdots, X_{n}$ 相互独立.
在离散随机变量场合, 如果对其任意 $n$ 个取值 $x_{1}, x_{2}, \cdots, x_{n}$, 有
$$
P\left(X_{1}=x_{1}, X_{2}=x_{2}, \cdots, X_{n}=x_{n}\right)=\prod_{i=1}^{n} P\left(X_{i}=x_{i}\right),
$$
则称 $X_{1}, X_{2}, \cdots, X_{n}$ 相互独立.
在连续随机变量场合, 如果对任意 $n$ 个实数 $x_{1}, x_{2}, \cdots, x_{n}$, 有
$$
p\left(x_{1}, x_{2}, \cdots, x_{n}\right)=\prod_{i=1}^{n} p_{i}\left(x_{i}\right),
$$
则称 $X_{1}, X_{2}, \cdots, X_{n}$ 相互独立.

### 习题3.2

**2.设二维随机变量$(X,Y)$的联合分布函数为**
$$
F(x, y)= \begin{cases}1-\mathrm{e}^{-\lambda_{1} x}-\mathrm{e}^{-\lambda_{2} y}+\mathrm{e}^{-\lambda_{1} x-\lambda_{2} y-\lambda_{12} \max |x, y|}, & x>0, y>0, \\ 0, & \text { 其他. }\end{cases}
$$
**试求$X$和$Y$各自的边际分布函数**

题目没有明确指出$\lambda_1, \lambda_2, \lambda_{12}$的正负性，假定它们都大于零

$x\le 0$时，$F_{X}(x)=0,\ p_{X}(x)=0$；

$x>0$时
$$
F_{X}(x)=1-\mathrm{e}^{-\lambda_{1} x}
$$
同理可得$Y$都边际分布函数。

**12.设随机变量 $(X, Y)$ 的联合密度函数为**
$$
p(x, y)= \begin{cases}3 x, & 0<x<1,0<y<x, \\ 0, & \text { 其他. }\end{cases}
$$
**试求:(1) 边际密度函数 $p_{x}(x)$ 和 $p_{y}(y)$; (2) $X$ 与 $Y$ 是否独立?**

(1)
$$
p_{X}(x)=\begin{cases}=\displaystyle\int_{0}^{x}3x\mathrm{d}y=3x^2,&  &0<x<1,\\
0,&  &其他.\end{cases}\\
p_{Y}(x)=\begin{cases}=\displaystyle\int_{y}^{1}3x\mathrm{d}x=\frac{3}{2}(1-y^2),&  &0<y<1,\\
0,&  &其他.\end{cases}
$$
(2)$X$ 与 $Y$ 不独立

**15.在长为$a$的线段的中点两边随机地各选取一点，求两点之间的距离小于$\displaystyle\frac{a}{3}$的概率**

设$X,Y$分别表示两点的横坐标，则$X\sim U(0,\displaystyle\frac{a}{2}),Y\sim U(\displaystyle\frac{a}{2},a)$，且两个事件相互独立
$$
p(x,y)=p_X(x)p_Y(y)=
\begin{cases}
\displaystyle \frac{4}{a^2},&&0<x<a/2,a/2<y<a,\\
0,&&其他.
\end{cases}
$$

$$
P(Y-X\le\frac{a}{3})=\int_{a/6}^{a/2}\int_{a/2}^{x+a/3}\frac{4}{a^2}\mathrm{d}y\mathrm{d}x=\int_{a/6}^{a/2}\frac{4}{a^2}(x-\frac{a}{6})\mathrm{d}x=\frac{2}{9}
$$

## 3.3 多维随机变量函数的分布

摘抄一些小结论

**泊松分布的可加性** 设随机变量$X\sim P(\lambda_1),Y\sim P(\lambda_2)$，且$X$与$Y$相互独立，则$Z=X+Y\sim P(\lambda_1+\lambda_2)$

**二项分布的可加性** 设随机变量$X\sim b(n,p), Y\sim b(m,p)$，且$X$与$Y$独立，则$Z=X+Y\sim b(n+m,p)$

**正态分布的可加性** 设随机变量$X\sim N(\mu_1,\sigma_1^2), Y\sim N(\mu_2,\sigma_2^2)$，且$X$与$Y$独立，则$Z=X+Y\sim N(\mu_1+\mu_2,\sigma_1^2+\sigma_2^2)$

**伽玛分布的可加性** 设随机变量 $X \sim G a\left(\alpha_{1}, \lambda\right), Y \sim G a\left(\alpha_{2}, \lambda\right)$, 且 $X$ 与 $Y$ 独立, 则 $Z=X+Y \sim G a\left(\alpha_{1}+\alpha_{2}, \lambda\right)$.

**令人感到迷惑的卡方分布的来源：** 设 $X_{1}, X_{2}, \cdots, X_{n}$ 是 $n$ 个相互独立同分布的**标准**正态变量,证明 其平方和 $Y=X_{1}^{2}+X_{2}^{2}+\cdots+X_{n}^{2}$ 服从自由度为 $n$ 的 $\chi^{2}$ 分布.

卡方分布和伽马分布，正态分布的关系都很紧密，是需要认真对待的一种分布。它的参数$n$表示独立的标准正态分布的个数，也可以叫它**自由度**



## 3.4 多维随机变量的特征数

### 3.4.1 多维随机变量函数的数学期望

定理 3.4.1 若二维随机变量 $(X, Y)$ 的分布用联合分布列 $P\left(X=x_{i}, Y=y_{j}\right)$ 或用联合密度函数 $p(x, y)$ 表示, 则 $Z=g(X, Y)$ 的数学期望为
$$
E(Z)=
\begin{cases}
\displaystyle\sum_{i} \sum_{j} g\left(x_{i}, y_{j}\right) P\left(X=x_{i}, Y=y_{j}\right),&& \text {在离散场合, } \\ 
\displaystyle\int_{-\infty}^{\infty} \int_{-\infty}^{\infty} g(x, y) p(x, y) \mathrm{d} x \mathrm{~d} y,&&\text{在连续场合.}
\end{cases}
$$
这里所涉及的数学期望都假设存在.

### 3.4.2 数学期望与方差的运算性质

**性质**  设 $(X, Y)$ 是二维随机变量, 则有
$$
E(X+Y)=E(X)+E(Y) .
$$
和的期望等于期望的和。

**性质** 若随机变量 $X$ 与 $Y$ **相互独立**, 则有
$$
E(X Y)=E(X) E(Y) .
$$
积的期望等于期望的乘积。

**性质** 若随机变量 $X$ 与 $Y$ **相互独立**, 则有
$$
\operatorname{Var}(X \pm Y)=\operatorname{Var}(X)+\operatorname{Var}(Y) \text {. }
$$
这条性质表明，**相互独立**的随机变量进行加减运算，**方差**只会逐个累积起来而**不会减少**



### 3.4.3 协方差

协方差是用来描述两个随机变量**相关程度**的特征数

**定义** 设$(X,Y)$是一个二维随机变量，如果$E[(X-E(X))(Y-E(Y))]$存在, 则称此数学期望为 $X$ 与 $Y$ 的协方差, 或称为 $X$ 与 $Y$ 的相关 (中心) 矩, 并记为
$$
\operatorname{Cov}(X, Y)=E[(X-E(X))(Y-E(Y))]
$$
 $X$ 与 $Y$ 的协方差是 $X$ 的偏差与 $Y$的偏差的乘积的数学期望。 
$$
\begin{align}
\operatorname{Cov}(X, Y)=&E[(X-E(X))(Y-E(Y))]\\
=&E(XY-XE(Y)-YE(X)+E(X)E(Y))\\
=&E(XY)-E(XE(Y))-E(YE(X))+E(E(X)E(Y))\\
=&E(XY)-2E(X)E(Y)+E(X)E(Y)\\
=&E(XY)-E(X)E(Y)
\end{align}
$$
**独立**的要求比相关更加严格：独立一定不相关，不相关不一定独立。

对于任意的两个随机变量：
$$
\begin{align}
Var(X\pm Y)&=E\{[(X\pm Y)-E(X\pm Y)]^2\}\\
&=E\{[(X-E(X))\pm(Y- E(Y))]^2\}\\
&=E[(X-E(X))^2]+E[(Y-E(Y))^2]\\
&\quad \pm 2E[(X-E(X))(Y-E(Y))]\\
&=Var(X)+Var(Y)\pm 2\operatorname{Cov}(X,Y)
\end{align}
$$
**协方差的一些性质**

* $\operatorname{Cov}(X, Y)=\operatorname{Cov}(Y, X)$
* $\operatorname{Cov}(X, a)=0$
* $\operatorname{Cov}(a X, b Y)=a b \operatorname{Cov}(X, Y)$
* $\operatorname{Cov}(X+Y, Z)=\operatorname{Cov}(X, Z)+\operatorname{Cov}(Y, Z)$



**相关系数的定义** 设 $(X, Y)$ 是一个二维随机变量, 且 $\operatorname{Var}(X)=\sigma_{X}^{2}>0, \operatorname{Var}(Y)=$ $\sigma_{Y}^{2}>0$. 则称
$$
\operatorname{Corr}(X, Y)=\frac{\operatorname{Cov}(X, Y)}{\sqrt{\operatorname{Var}(X)} \sqrt{\operatorname{Var}(Y)}}=\frac{\operatorname{Cov}(X, Y)}{\sigma_{X} \sigma_{Y}}
$$
为 $X$ 与 $Y$ 的 (线性) 相关系数.

$-1\le\operatorname{Corr}(X, Y)\le1$



**相关系数说明了什么？**

**相关系数描述了两个随机变量之间线性关系的强弱**，

> - 相关系数 $\operatorname{Corr}(X, Y)$ 刻画了 $X$ 与 $Y$ 之间的线性关系强弱, 因此也常称其 为 “线性相关系数”.
> - 若 $\operatorname{Corr}(X, Y)=0$, 则称 $X$ 与 $Y$ 不相关. 不相关是指 $X$ 与 $Y$ 之间没有**线性关系**, 但 $X$ 与 $Y$ 之间可能有其他的函数关系, 譬如平方关系、对数关系等.
> - 若 $\operatorname{Corr}(X, Y)=1$, 则称 $X$ 与 $Y$ 完全**正相关**; 若 $\operatorname{Corr}(X, Y)=-1$, 则称 $X$ 与 $Y$ 完全负相关.

### 3.4.5 随机向量的数学期望向量与协方差矩阵

**定义** 记 $n$ 维随机向量为 $X=\left(X_{1}, X_{2}, \cdots, X_{n}\right)^{\prime}$, 若其每个分量的数学期望都存在, 则称
$$
E(X)=\left(E\left(X_{1}\right), E\left(X_{2}\right), \cdots, E\left(X_{n}\right)\right)^{\prime}
$$
为 $n$ 维随机向量 $X$ 的数学期望向量, 简称为 $X$ 的数学期望, 而称
$$
\begin{aligned}
& E\left[(\boldsymbol{X}-E(\boldsymbol{X}))(\boldsymbol{X}-E(\boldsymbol{X}))^{\prime}\right] \\
=&\left(\begin{array}{cccc}
\operatorname{Var}\left(X_{1}\right) & \operatorname{Cov}\left(X_{1}, X_{2}\right) & \cdots & \operatorname{Cov}\left(X_{1}, X_{n}\right) \\
\operatorname{Cov}\left(X_{2}, X_{1}\right) & \operatorname{Var}\left(X_{2}\right) & \cdots & \operatorname{Cov}\left(X_{2}, X_{n}\right) \\
\vdots & \vdots & & \vdots \\
\operatorname{Cov}\left(X_{n}, X_{1}\right) & \operatorname{Cov}\left(X_{n}, X_{2}\right) & \cdots & \operatorname{Var}\left(X_{n}\right)
\end{array}\right)
\end{aligned}
$$
为该随机向量的**方差-协方差矩阵**, 简称**协方差阵**, 记为 $\operatorname{Cov}(\boldsymbol{X})$.

重要性质：**协方差阵是对称非负定阵**

### 习题3.4

**3.从数字$0,1,\cdots,n$中任取两个不同的数字，求这两个数字之差的绝对值的数学期望**

设随机变量$X$表示取出两个不同数字的差的绝对值，$X$的可能取值有$0,1,\cdots,n$
$$
P(X=i)=2\times\frac{n+1-i}{n(n+1)}\\
E(X)=2\times\sum_{i=0}^{n}i\times\frac{n+1-i}{n(n+1)}=\frac{n+2}{3}
$$
这道题最初做的时候，$P(X=i)$ 并没有乘以2，所以最后结果正好是答案的$1/2$，后来想了想，抽取两个数字，应该要考虑抽取次序，数字之差的绝对值相同的一对数字有两种顺序，所以$P(X=i)$ 要乘以2

**11. 设随机变量 $(X, Y)$ 的联合密度函数为**
$$
p(x, y)= \begin{cases}x\left(1+3 y^{2}\right) / 4, & 0<x<2,0<y<1, \\ 0, & \text { 其他. }\end{cases}
$$
**试求 $E(Y / X)$.**
$$
E(Y/X)=\int_{0}^{1}\int_{0}^{2}\frac{y}{x}\frac{x(1+3y^2)}{4}\mathrm{d}x\mathrm{d}y=\frac{5}{8}
$$
**23. 将一枚硬币重复抛 $n$ 次, 以 $X$ 和 $Y$ 分别表示正面向上和反面向上的次数, 试求 $X$ 和 $Y$ 的协方差和相关系数.**

$X\sim b(n,0.5),\ Y\sim b(n,0.5)$，且$X+Y=n$

$E(X)=\displaystyle\frac{n}{2},\ E(Y)=\displaystyle\frac{n}{2}$
$$
\begin{align}
E(XY)&=\sum_{y=0}^{n} \sum_{x=0}^{n} xyC_{n}^{x}0.5^x0.5^{n-x}C_{n}^{y}0.5^y0.5^{n-y}\\
&=\sum_{x=0}^{n}x(n-x)C_{n}^{x}0.5^x0.5^{n-x}C_{n}^{n-x}0.5^x0.5^{n-x}\\
&=(\frac{1}{4})^n\frac{n(n-1)(2n-2)!}{(n-1)!(n-1)!}
\end{align}
$$
~~$\operatorname{Cov}(X,Y)=E(XY)-E(X)E(Y)=\displaystyle(\frac{1}{4})^n\frac{n(n-1)(2n-2)!}{(n-1)!(n-1)!}-\frac{n^2}{4}$~~

上面的计算过程出错了，我还没找出来哪里错了，下面是正确答案

$\operatorname{Cov}(X,Y)=\operatorname{Cov}(X,n-X)=-\displaystyle\operatorname{Cov}(X,X)=-\frac{n}{4}$

$\operatorname{Corr}(X,Y)=\displaystyle\frac{\operatorname{Cov}(X,Y)}{\sigma_1\sigma_2}=\frac{-\frac{n}{4}}{\frac{\sqrt{n}}{2}\frac{\sqrt{n}}{2}}=-1$

这表示$X$与$Y$完全负相关，反映了$X+Y=n$这一关系

## 3.5 条件分布与条件期望

### 3.5.1 条件分布

在二维随机变量的情况下，指定其中一个随机变量的值，求出另一个变量的分布，就是条件分布。

**一、离散随机变量的条件分布**

**定义** 对一切使 $P\left(Y=y_{i}\right)=p_{. j}=\displaystyle\sum_{i=1}^{\infty} p_{i j}>0$ 的 $y_{j}$, 称 
$$
p_{i | j}=P\left(X=x_{i} \mid Y=y_{j}\right)=\frac{P\left(X=x_{i}, Y=y_{j}\right)}{P\left(Y=y_{j}\right)}=\frac{p_{i j}}{p_{. j}}, \quad i=1,2, \cdots
$$
为给定 $Y=y_{j}$ 条件下 $X$ 的条件分布列.
同理, 对一切使 $P\left(X=x_{i}\right)=p_{i}=\sum_{j=1}^{\infty} p_{i j}>0$ 的 $x_{i}$, 称 
$$
p_{j | i}=P\left(Y=y_{j} \mid X=x_{i}\right)=\frac{P\left(X=x_{i}, Y=y_{j}\right)}{P\left(X=x_{i}\right)}=\frac{p_{i j}}{p_{i} \cdot}, \quad j=1,2, \cdots
$$
为给定 $X=x_{i}$ 条件下 $Y$ 的条件分布列.

**二、连续随机变量的条件分布**

**定义** 对一切使 $p_{Y}(y)>0$ 的 $y$, 给定 $Y=y$ 条件下 $X$ 的条件分布函数和条件密度函数分别为
$$
\begin{aligned}
&F(x \mid y)=\int_{-\infty}^{x} \frac{p(u, y)}{p_{Y}(y)} \mathrm{d} u, \\
&p(x \mid y)=\frac{p(x, y)}{p_{Y}(y)} .
\end{aligned}
$$
同理对一切使 $p_{X}(x)>0$ 的 $x$, 给定 $X=x$ 条件下 $Y$ 的条件分布函数和条件密度函数分别为
$$
F(y \mid x)=\int_{-\infty}^{y} \frac{p(x, v)}{p_{X}(x)} \mathrm{d} v,\\
p(y \mid x)=\frac{p(x, y)}{p_{X}(x)} .
$$
**三、连续场合的全概率公式和贝叶斯公式**

全概率公式
$$
p_X(x)=\int_{-\infty}^{\infty}p_Y(y)p(x|y)\mathrm{d}y\\
p_Y(y)=\int_{-\infty}^{\infty}p_X(x)p(y|x)\mathrm{d}x
$$
贝叶斯公式
$$
p(x|y)=\frac{p_X(x)p(y|x)}{\displaystyle\int_{-\infty}^{\infty}p_X(x)p(y|x)\mathrm{d}x}\\
p(y|x)=\frac{p_Y(y)p(x|y)}{\displaystyle\int_{-\infty}^{\infty}p_Y(y)p(x|y)\mathrm{d}y}
$$

### 3.5.2 条件数学期望

这是条件分布的数学期望
$$
E(X \mid Y=y)=\left\{\begin{array}{cc}\displaystyle\sum_{i} x_{i} P\left(X=x_{i} \mid Y=y\right), & (X, Y) \text { 为二维离散随机变量, } \\ 
\displaystyle\int_{-\infty}^{\infty} x p(x \mid y) \mathrm{d} x, & (X, Y) \text { 为二维连续随机变量. }\end{array}\right.\\
E(Y \mid X=x)=\left\{\begin{array}{cc}\displaystyle\sum_{j} y_{j} P\left(Y=y_{j} \mid X=x\right), & (X, Y) \text { 为二维离数随机变量, } \\ 
\displaystyle\int_{-\infty}^{\infty} y p(y \mid x) \mathrm{d} y, & (X, Y) \text { 为二维连续随机变量. }\end{array}\right.
$$
$E(X \mid Y=y)$是$y$的函数，记为$g(y)$。

$g(y)$的取值也是一个随机变量，记为$g(Y)$，$g(Y)=E(X|Y)$



**重期望公式** 设$(X,Y)$是二维随机变量，且$E(X)$存在，则
$$
E(X)=E(E(X|Y))
$$
利用这个结论，我们可以把很大的样本空间先分成一些小的样本空间，计算每个小样本空间内某个随机变量的期望，最后求不同小样本空间该随机变量期望的期望，就能得到在整个样本空间中该随机变量的期望了。

重期望公式的具体使用如下:
(1) 如果 $Y$ 是一个离散随机变量, 则
$$
E(X)=\sum_{j} E\left(X \mid Y=y_{j}\right) P\left(Y=y_{j}\right) .
$$
(2) 如果 $Y$ 是一个连续随机变量, 则 
$$
E(X)=\int_{-\infty}^{\infty} E(X \mid Y=y) p_{y}(y) \mathrm{d} y .
$$
**随机个随机变量和的数学期望** 设 $X_{1}, X_{2}, \cdots$ 为一列独立同分布的随机变量，随机变量 $N$ 只取正整数值，且 $N$ 与 $\left\{X_{n} \right\}$ 独立，则
$$
E\left(\sum_{i=1}^{N} X_{i}\right)=E\left(X_{1}\right) E(N) .
$$
此结论可以解决的问题举例：

(1)设一天内到达某商场的顾客数 $N$ 是仅取非负整数值的随机变量, 且 $E(N)=35000$. 又设进入此商场的第 $i$ 个顾客的购物金额为 $X_{i}$, 可以认为诸 $X_{i}$ 是独立同分布的随机变量, 且 $E\left(X_{i}\right)=82\left(\right.$ 元). 假设 $N$ 与 $X_{i}$ 相互独立是合理的, 则此商场一天的平均营业额为
$$
E\left(\sum_{i=0}^{N} X_{i}\right)=E\left(X_{1}\right) E(N)=82 \times 35000=287 \text { (万元) , }
$$
其中 $X_{0}=0$.
(2) 一只昆虫一次产卵数 $N$ 服从参数为 $\lambda$ 的泊松分布，每个卵能成活的概率是 $p$, 可设 $X_{i}$ 服从 0-1 分布, 而 $\left\{X_{i}=1\right\}$ 表示第 $i$ 个卵成活, 则一只昆虫一次产卵后的平均成活卵数为
$$
E\left(\sum_{i=1}^{N} X_{i}\right)=E\left(X_{1}\right) E(N)=\lambda p .
$$

# 第四章 大数定律与中心极限定理

## 4.1 随机变量序列的两种收敛性

依概率收敛和依分布收敛

### 4.1.1 依概率收敛

**定义** 设 $\left\{X_{n}\right\}$ 为一随机变量序列, $X$ 为一随机变量, 如果对任意的 $\varepsilon>0$, 有
$$
P\left(\left|X_{n}-X\right| \geqslant \varepsilon\right) \rightarrow 0(n \rightarrow \infty),
$$
则称序列 $\left\{X_{n}\right\}$ 依概率收敛于 $X$, 记作 $X_{n} \stackrel{P}{\longrightarrow} X$.

**定理（依概率收敛的运算性质）**  设 $\left\{X_{n}\right\},\left\{Y_{n} \}\right.$ 是两个随机变量序列, $a, b$ 是两个常数. 如果
$$
X_{n} \stackrel{P}{\longrightarrow} a, \quad Y_{n} \stackrel{P}{\longrightarrow} b,
$$
则有 (1) $X_{n} \pm Y_{n} \stackrel{P}{\longrightarrow} a \pm b$;
(2) $X_{n} \times Y_{n} \stackrel{P}{\longrightarrow} a \times b$;
(3) $X_{n} \div Y_{n} \stackrel{P}{\longrightarrow} a \div b(b \neq 0)$.

### 4.1.2 按分布收敛、弱收敛

在引出按分布收敛的弱收敛定义之前，书里先讨论了要求分布函数[逐点收敛](https://baike.baidu.com/item/逐点收敛/22781982)于某一函数为什么太严格了（会破坏分布函数的右连续性）

**定义** 设随机变量 $X, X_{1}, X_{2}, \cdots$ 的分布函数分别为 $F(x), F_{1}(x)$, $F_{2}(x), \cdots$. 若对 $F(x)$ 的任一**连续点 **$x$, 都有
$$
\lim_{n\to \infty}F_n(x)=F(x)
$$
则称 $\left\{F_{n}(x)\right\}$ **弱收敛**于 $F(x)$, 记作
$$
F_{n}(x) \stackrel{W}{\longrightarrow} F(x)
$$
也称 $\left\{X_{n}\right\}$ 按分布收敛于 $X$, 记作
$$
X_{n} \stackrel{L}{\longrightarrow} X .
$$
**定理** $X_{n} \stackrel{P}{\longrightarrow} X \Longrightarrow X_{n} \stackrel{L}{\longrightarrow} X$.

概率收敛能推出分布收敛



**定理** 若 $c$ 为常数, 则 $X_{n} \stackrel{P}{\longrightarrow} c$ 的**充要**条件是: $X_{n} \stackrel{L}{\longrightarrow} c$.

## 4.2 特征函数

### 4.2.1 定义

定义 4.2.1 设 $X$ 是一个随机变量, 称
$$
\varphi(t)=E\left(\mathrm{e}^{i t x}\right)=
\begin{cases}
\begin{align}
\int_{-\infty}^{\infty}\mathrm{e}^{itx}p(x)\mathrm{d}x,\quad-\infty<t<\infty,连续场合,\\
\sum_{k=1}^{\infty}\mathrm{e}^{itx_k}p(x_k),\quad-\infty<t<\infty,离散场合.
\end{align}
\end{cases}
$$
为 $X$ 的特征函数.

常用分布的特征函数

| 分布                                                         | 特征函数                                                     |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| 单点分布$P(X=a)=1$                                           | $\varphi(t)=\mathrm{e}^{ita}$                                |
| 0-1分布$P(X=x)=p^x(1-p)^{1-x},x=0,1$                         | $\varphi(t)=p\mathrm{e}^{it}+1-p$                            |
| 泊松分布$P(X=k)=\displaystyle\frac{\lambda^k}{k!}\mathrm{e}^{-\lambda},k=0,1,\cdots$ | $\varphi(t)=\mathrm{e}^{\lambda(\mathrm{e}^{it}-1)}$         |
| 均匀分布$U(a,b)$                                             | $\varphi(t)=\displaystyle\frac{\mathrm{e}^{itb}-\mathrm{e}^{ita}}{it(b-a)}$ |
| 标准正态分布$N(0,1)$                                         | $\varphi(t)=\mathrm{e}^{-\frac{t^2}{2}}$                     |
| 指数分布$Exp(\lambda)$                                       | $\varphi(t)=(1-\displaystyle\frac{it}{\lambda})^{-1}$        |
| 二项分布$b(n,p)$                                             | $\varphi(t)=(p\mathrm{e}^{it}+q)^n,\ q=1-p$                  |
| 正态分布$N(\mu,\sigma^2)$                                    | $\varphi(t)=\mathrm{exp}\{i\mu t-\displaystyle\frac{\sigma^2t^2}{t}\}$ |
| 伽马分布$Ga(n,\lambda)$                                      | $\varphi(t)=(1-\displaystyle\frac{it}{\lambda})^{-n}$        |
| $\chi^{2}(n)$ 分布  $\chi^{2}(n)=Ga(n,\lambda)$              | $\varphi(t)=(1-2 i t)^{-1 / 2} $                             |

### 4.2.2 特征函数的性质

* $|\varphi(t)| \leqslant \varphi(0)=1 . \quad$
* $\varphi(-t)=\overline{\varphi(t)}$, 其中 $\overline{\varphi(t)}$ 表示 $\varphi(t)$ 的共轭.
* 若 $Y=a X+b$, 其中 $a, b$ 是常数, 则

$$
\varphi_{Y}(t)=e^{i t b} \varphi_{X}(a t)
$$

* 独立随机变量和的特征函数为每个随机变量的特征函数的积, 即设 $X$ 与 $Y$ 相互独立, 则

$$
\varphi_{X+Y}(t)=\varphi_{X}(t) \varphi_{Y}(t) .
$$

* 若 $E\left(X^{l}\right)$ 存在, 则 $X$ 的特征函数 $\varphi(t)$ 可 $l$ 次求导, 且对 $1 \leqslant k \leqslant l$, 有

$$
\varphi^{(k)}(0)=\mathrm{i}^{k} E\left(X^{k}\right) .
$$

$E(X)=\displaystyle\frac{\varphi^{'}(0)}{i},\ Var(X)=E(X^2)-(E(X)^2)=-\varphi^{''}(0)+(\varphi^{'}(0))^2$

* 随机变量$X$的特征函数$\varphi(t)$在$(-\infty,\infty)$上一致连续

* **非负定性** 随机变量 $X$ 的特征函数 $\varphi(t)$ 是非负定的, 即对任意正整数 $n$ 及 $n$ 个实数 $t_{1}, t_{2}, \cdots, t_{n}$ 和 $n$ 个复数 $z_{1}, z_{2}, \cdots, z_{n}$, 有
	$$
	\sum_{k=1}^{n} \sum_{j=1}^{n} \varphi\left(t_{k}-t_{j}\right) z_{k} \bar{z}_{j} \geqslant 0 .
	$$

### 4.2.3 特征函数唯一决定分布函数

$$
\varphi(t)=\int_{-\infty}^{\infty} e^{i t x} p(x) \mathrm{d} x,\\
p(x)=\frac{1}{2 \pi} \int_{-\infty}^{\infty} e^{-i t x} \varphi(t) \mathrm{d} t .
$$

密度函数$\stackrel{傅立叶变换}{\longrightarrow}$特征函数

特征函数$\stackrel{傅立叶逆变换}{\longrightarrow}$密度函数

**定理** 分布函数序列 $\left\{F_{n}(x)\right\}$ 弱收敛于分布函数 $F(x)$ 的充要条件是 $\left\{F_{n}(x) \}\right.$ 的特征函数序列 $\left.\{ \varphi_{n}(t)\right\}$ 收敛于 $F(x)$ 的特征函数 $\varphi(t)$.



## 4.3 大数定律

### 4.3.1 伯努利大数定律

**定理 (伯努利大数定律)** 设 $s_{n}$ 为 $n$ 重伯努利试验中事件 $A$ 发生的次 数, $p$ 为每次试验中 $A$ 出现的概率, 则对任意的 $\varepsilon>0$, 有
$$
\lim _{n \rightarrow \infty} P\left(\left|\frac{s_{n}}{n}-p\right|<\varepsilon\right)=1 \text {. }
$$

### 4.3.2 常用的几个大数定律

**定理 (切比雪夫大数定律 )** 设 $\left\{X_{n} \}\right.$ 为一列**两两不相关**的随机变量序列, 若每个 $X_{i}$ 的方差存在, 且有共同的上界, 即 $\operatorname{Var}\left(X_{i}\right) \leqslant c, i=1,2, \cdots$, 则 $\left|X_{n}\right|$ 服从大数定律, 即对任意的 $\varepsilon>0,$ 下式成立.
$$
\lim _{n \rightarrow\infty} P\left(\left|\frac{1}{n} \sum_{i=1}^{n} X_{i}-\frac{1}{n} \sum_{i=1}^{n} E\left(X_{i}\right)\right|<\varepsilon\right)=1 .
$$
切比雪夫大数定律只要求变量**不相关**和方差有同上界，对分布没有要求。



**定理（马尔可夫大数定律）** 对随机变量序列$\{X_n\}$，如果$\displaystyle\frac{1}{n^2}\operatorname{Var}(\sum_{i=1}^{n}X_i)\rightarrow 0$（马尔可夫条件），则$\{X_n\}$​服从大数定律，即对任意的 $\varepsilon>0,$ 下式成立.
$$
\lim _{n \rightarrow\infty} P\left(\left|\frac{1}{n} \sum_{i=1}^{n} X_{i}-\frac{1}{n} \sum_{i=1}^{n} E\left(X_{i}\right)\right|<\varepsilon\right)=1 .
$$


**定理 (辛钦大数定律)** 设 $\left\{X_{n}\right\}$ 为一**独立同分布**的随机变量序列, 若 $X_{i}$ 的数学期望存在, 则 $\left\{X_{n}\right\}$ 服从大数定律, 即对任意的 $\varepsilon>0,$​ 下式成立.
$$
\lim _{n \rightarrow\infty} P\left(\left|\frac{1}{n} \sum_{i=1}^{n} X_{i}-\frac{1}{n} \sum_{i=1}^{n} E\left(X_{i}\right)\right|<\varepsilon\right)=1 .
$$

## 4.4 中心极限定理

大数定律讨论的是**随机变量的算术平均**在什么条件下依概率收敛于其**均值的算术平均**。

中心极限定理讨论的是**随机变量和**的分布在什么条件下收敛于**正态分布**。

### 4.4.2 独立同分布下的中心极限定理

**定理  (林德伯格-莱维 (Lindeberg-Lévy) 中心极限定理)** 设 $\left\{X_{n}\right\}$ 是 **独立同分布**的随机变量序列, 且 $E\left(X_{i}\right)=\mu, \operatorname{Var}\left(X_{i}\right)=\sigma^{2}>0$ 存在, 若记
$$
Y_{n}^{*}=\frac{X_{1}+X_{2}+\cdots+X_{n}-n \mu}{\sigma \sqrt{n}},
$$
则对任意实数 $y$, 有
$$
\lim _{n \rightarrow \infty} P\left(Y_{n}^{*} \leqslant y\right)=\Phi(y)=\frac{1}{\sqrt{2 \pi}} \int_{-\infty}^{y} e^{-\frac{t^{2}}{2}} \mathrm{~d} t .
$$

### 4.4.4 独立不同分布下的中心极限定理

**定理(林德伯格中心极限定理)** 设**独立**随机变量序列 $\left\{X_{n}\right\}$ 满足$\displaystyle\lim _{n \rightarrow \infty} \frac{1}{\tau^{2} B_{n}^{2}} \sum_{i=1}^{n} \int_{\left|x-\mu_{i}\right|>\tau B_{n}}\left(x-\mu_{i}\right)^{2} p_{i}(x) \mathrm{d} x=0$（林德伯格条件）, 则对任意的 $x$, 有
$$
\lim _{n \rightarrow \infty} P\left(\frac{1}{B_{n}} \sum_{i=1}^{n}\left(X_{i}-\mu_{i}\right) \leqslant x\right)=\frac{1}{\sqrt{2 \pi}} \int_{-\infty}^{x} e^{-t^{2} / 2} \mathrm{~d} t .
$$
其中，$\tau>0$是任意正实数，$B_n = \sqrt{\sigma_1^2 + \sigma_2^2 + \cdots + \sigma_n^2}$，$\mu_i$是$X_i$的数学期望值。



**定理(李雅普诺夫中心极限定理)** 设 $\left\{X_{n}\right\}\left\{X_{n}\right\}$ 为独立随机变量序列, 若存 在 $\delta>0$, 满足
$$
\lim _{n \rightarrow \infty} \frac{1}{B_{n}^{2+\delta}} \sum_{i=1}^{n} E\left(\left|X_{i}-\mu_{i}\right|^{2+\delta}\right)=0
$$
则对任意的 $x$, 有
$$
\lim _{n \rightarrow \infty} P\left(\frac{1}{B_{n}} \sum_{i=1}^{n}\left(X_{i}-\mu_{i}\right) \leqslant x\right)=\frac{1}{\sqrt{2 \pi}} \int_{-\infty}^{x} \mathrm{e}^{-t^{2} /2} \mathrm{~d} t .
$$
其中 $\mu_{i}$ 与 $B_{n}$ 如前所述.

