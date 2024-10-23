---
layout: post
title:  分式线性变换
date:   2024-08-01 15:30:00 +0300
tags:   保角变换
description:   变换
---

# [简介](#简介)

“分式线性变换”(Fractional Linear Transformations)也有称“线性分式变换”(Linear Fractional Transformations)或者“莫比乌斯变换”(Mobius Transformations), 其形式如下：     

$$w=f(z)=\frac{az+b}{cz+d}$$ 

其中$a,b,c,d\in \mathbb{C}$且：

$$ad-bc \ne 0$$

其是复平面上的保角变换。其可以把圆弧映射为圆弧，称其具有保圆的性质。在处理全景图上的色块映射时具有较好的效果。     

通过一些计算可以证明分式线性变换的复合和逆变换还是分式线性变换。具体如下：   

$$T_{1}(z)=\frac{a_{1}z+b_{1}}{c_{1}z+d_{1}}$$   

$$T_{2}(z)=\frac{a_{2}z+b_{2}}{c_{2}z+d_{2}}$$   

其复合变换:   

$$(T_{1}\circ T_{2})(z)=\frac{az+b}{cz+d}$$

$$
\left (
\begin{array}{cc}
a_{1} & b_{1} \\
c_{1} & d_{1} \\
\end{array}
\right )
\left (
\begin{array}{cc}
a_{2} & b_{2} \\
c_{2} & d_{2} \\
\end{array}
\right )
=
\left (
\begin{array}{cc}
a & b \\
c & d \\
\end{array}
\right )
$$

如果$c=0$则这个变换是**线性变换**。

# [定理](#(定理))

定理：    

给两组数量为3的点集$\{z_{0},z_{1},z_{2}\},\{w_{0},w_{1},w_{2}\}\subset \mathbb{C} \cup\{\infty\}$其之间必存在唯一的分式线性变换(FLT)使得$f(z_j)=w_{j}, j=0,1,2$成立。

证明：这里就不证明了，下面直接给出公式。     

公式一，将任意3点$p_{0}, p_{1}, p_{2}$分别映射到$0, \infin, 1$：    

$$
T=
\left (
\begin{array}{cc}
p_{2}-p_{1} & -p_{0}*(p_{2}-p_{1})\\
p_{2}-p_{0} & -p_{1}*(p_{2}-p_{0}) \\
\end{array}
\right )
$$

公式二，将$0, \infin, 1$分别映射到任意3点$p_{0}, p_{1}, p_{2}$, 显然与一互逆：   

$$
T^{-1} = 
\left (
\begin{array}{cc}
-p_{1}*(p_{2}-p_{0}) & p_{0}*(p_{2}-p_{1})\\
-(p_{2}-p_{0}) & p_{2}-p_{1} \\
\end{array}
\right ) / det(T)
$$

$$
det(T) = T(0,0) * T(1,1) - T(0,1)*T(1,0)
$$

通过这连个变换的复合，可以得到目标变换：  

$$
T=T_{z}\circ T_{w}^{-1}
$$


