---
layout: post
title:  三角形重心坐标系
date:   2024-07-26 15:30:00 +0300
tags:   PrimitiveGeometry
description:  简单几何
---


# [重心坐标系](#重心坐标系)
三角形的三个顶点$A,B,C$唯一的确定了一个空间中的平面，平面法向：   

$$n=(B-A) \times (C-A)=b \times c$$

这个平面上的所有点，都可以表示为下面的形式：    

$$q=A + \beta \cdot (B-A) + \gamma \cdot (C-A) = A + \beta \cdot b + \gamma \cdot c$$

整理一下得到

$$q = (1-\beta - \gamma) \cdot A + \beta \cdot B + \gamma \cdot C$$

令

$$\alpha = 1 - \beta - \gamma$$

![]({{ site.baseurl }}/images/triangleBcoords-001.jpg)  

当点$q$位于三角形内部时:

$$\beta \ge 0$$ 
$$\gamma \ge 0$$ 
$$\alpha = 1- \beta-\gamma \ge 0$$

# [重心坐标求解](#重心坐标求解)

对任意一点$q$其可表示为：

$$q = A + \beta \cdot b + \gamma \cdot c$$

那么:

$$q-A= \beta \cdot b + \gamma \cdot c$$

令:

$$ a = q - A $$

则等式变为:

$$a = \beta \cdot b + \gamma \cdot c$$

等式两侧同时乘$b,c$则：

$$a \cdot b = \beta \cdot b \cdot b + \gamma \cdot c \cdot b  \quad (1)$$
$$a \cdot c = \beta \cdot b \cdot c + \gamma \cdot c \cdot c \quad (2)$$

从$(1)$式可得：

$$\beta = \frac{a \cdot b - \gamma \cdot c \cdot b}{b \cdot b} \quad (3)$$

将$(3)$式代入$(2)$式，可得：

$$\gamma = \frac{a \cdot c \cdot b \cdot b - a \cdot b \cdot b \cdot c}{c \cdot c \cdot b \cdot b - c \cdot b \cdot b \cdot c} \quad (4)$$

将$(4)$d带入$(3)$式，可得：

$$\beta = \frac{a \cdot b \cdot c \cdot c - a \cdot c \cdot c \cdot b}{c \cdot c \cdot b \cdot b - c \cdot b \cdot b \cdot c}$$






