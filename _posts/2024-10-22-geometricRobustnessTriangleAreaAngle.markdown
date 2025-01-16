---
layout: post
title:  三角形面积与内角计算
date:   2024-10-21 15:30:00 +0300
tags:   PrimitiveGeometry
description:  简单几何
---

# [简介](#简介)

当遇到针状三角形时，其面积与内角的计算受浮点数计算舍入误差影响较大，这里对其精度进行讨论。

![]({{ site.baseurl }}/images/triangleAreaAngle-001.jpg)  

# [三角形面积](#三角形面积)

1. 对边长进行排序使$a>b>c$.     
2. 判定$c-(a-b)<0$是否成立。    
3. 计算面积，公式如下:     

$$
\bigtriangleup(a,b,c):=\frac{1}{4}\sqrt{\big((a+(b+c))(c-(a-b))(c+(a-b))(a+(b-c))\big)}
$$

公式的括号很重要，不要移除！！！

# [三角形内角](#三角形内角)

1. 保证$a>b$.   
2. 计算中间量$\mu$, 过程如下：
```
if b>=c>=0:
    u := c-(a-b)
elseif c>b>=0:
    u := b-(a-c)
else:
    not except condition
```
3. 计算角度:

$$
C(a, b, c) := 2 \arctan \bigg(\sqrt{\Big(((a-b)+c)\mu / \big((a+(b+c))((a-c)+b)\big)\Big)}\bigg)
$$

公式的括号很重要，不要移除！！！

# [样例](样例)

![]({{ site.baseurl }}/images/robustTriangleAreaAngle-001.jpg)    

![]({{ site.baseurl }}/images/robustTriangleAreaAngle-002.jpg)


