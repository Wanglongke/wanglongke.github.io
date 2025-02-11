---
layout: post
title:  泊松图像编辑
date:   2025-01-15 15:30:00 +0300
tags:   ComputerVision
description:  泊松方程
---

# [简介](#简介)

参考论文：Pérez Patrick, Gangnet Michel, Blake Andrew. 2003. Poisson image editing. ACM Transactions on Graphics.    

# [引导插值](#引导插值)

由于插值问题对于图像的各个通道的过程是独立的，一致的，这里用单通道做示例。

![]({{ site.baseurl }}/images/poissionImageEdit-001.png)

$S$ 是 $\mathbb{R}^2$ 上的封闭子集，就是图像空间。     
$\Omega$ 是 $S$ 上的一个封闭区域，其边界为 $\partial \Omega$.     

$f^{\star}$ 是$S-\Omega$ 上的已知数值函数。   

$f$ 是 $\Omega$ 上的未知数值函数。    

$\mathbf{v}$ 是 $\Omega$ 上的向量场。  

最简单的插值就是"membrane"插值，其是下面问题的解：   

$$\min_{f} \int\int_{\Omega}\lvert\nabla f\rvert^{2} \ \mathrm{with} \ f\lvert_{\partial \Omega} = f^{\star}\lvert_{\partial \Omega} \quad (1)$$

其中 $\nabla .=[\frac{\partial .}{\partial x}, \frac{\partial .}{\partial y}]$ 表示梯度。      

其Euler-Lagrange方程的形式如下:   

$$\Delta f=0 \ \mathrm{over} \ \Omega \ \mathrm{with} \ f|_{\partial \Omega}=f^{\star}|_{\partial \Omega} \quad (2)$$

其中 $\Delta .=[\frac{\partial^{2} .}{\partial x^{2}}, \frac{\partial^{2} .}{\partial y^{2}}]$ 表示Laplacian运算。          

引导向量场 $\mathbf{v}$ 的使用将公式(1)的问题扩展为下面问题:      

$$\min_{f}\int\int_{\Omega} |\nabla f-\mathbf{v}|^{2} \ \mathrm{with} \ f|_{\partial \Omega}=f^{*}|_{\partial \Omega} \quad (3)$$    

其可转换为具有Dirichlet边界的Poisson问题：    

$$\Delta f = \mathrm{div} \mathbf{v} \ \mathrm{over} \ \Omega , \ \mathrm{with} \ f|_{\partial \Omega}= f^{*}|_{\partial \Omega} \quad (4)$$

其中 $\mathrm{div} \mathbf{v}=\frac{\partial u}{\partial x} + \frac{\partial v}{\partial y}$ 是向量场 $\mathbf{v}=(u,v)$的散度。     

对于彩色图像的三个通道可以使用公式(4)独立求解。    

当向量场 $\mathbf{v}$ 为某种数值场的梯度，则问题(4)可以被简化为以下形式:    

$$\Delta \tilde{f}=0 \ \mathrm{over} \ \Omega, \ \tilde{f}|_{\partial \Omega}=(f^{\star}-g)|_{\partial \Omega} \quad (5)$$   

其中 $f=g+\tilde{f}$.    

# [离散化泊松求解](#离散化泊松求解)

对于 $S$ 上的任一像素 $p$ 其四邻域(上下左右)表示为 $N_{p}$ 其邻域对表示为 $<p,q>,\ q\in N_{p}$ . 区域 $\Omega$ 的边界表示为 $\partial \Omega = \{p \in S \backslash \Omega : N_{p} \cap \Omega \ne \empty \}$ . 用 $f_{p}$ 表示 $f$ 位于 $p$ 处的值。    

问题点的离散形式如下：

$$\min_{f|_{\Omega}} \sum_{<p,q>,p\in\Omega} (f_{p}-f_{q}-v_{pq})^{2}, \ \mathrm{with} f_{q}=f^{\star}_{q}, \ \mathrm{for} \ \mathrm{all} \ q \in \partial \Omega \quad (6)$$

其中 

$$v_{pq}=\mathbf{v}(\frac{p+q}{2}) \cdot \vec{pq} \ $$ 

其解必定满足以下线性方程:    

$$\mathrm{for} \ \mathrm{all} \ p\in\Omega ,\ |N_{p}|f_{p}-\sum_{q\in N_{p}\cap\Omega} f_{q} = |N_{p}|f_{p}-\sum_{q\in N_{p}\cap\partial\Omega} f^{\star}_{q} + \sum_{q\in N_{p}}v_{pq}   \quad (7)$$

其中$|N_{p}|$表示其邻域个数。当位于图像边界时，其会小于4; 位于图像内部时，其会等于4.      

为了方便理解，这里将公式(7)最后一项展开：   

$$\sum_{q\in N_{p}}v_{pq} = \mathbf{v}(\frac{p+q_{0}}{2})*\vec{pq_{0}} + 
                            \mathbf{v}(\frac{p+q_{1}}{2})*\vec{pq_{1}} + 
                            \mathbf{v}(\frac{p+q_{2}}{2})*\vec{pq_{2}} +
                            \mathbf{v}(\frac{p+q_{3}}{2})*\vec{pq_{3}} \\
                          = (\frac{u_{p}+u_{q_{0}}}{2}, \frac{v_{p}+v_{q_{0}}}{2}) * (\ 1, \ 0) + \\
                            \quad (\frac{u_{p}+u_{q_{1}}}{2}, \frac{v_{p}+v_{q_{1}}}{2}) * (\ 0, \ 1) + \\ 
                            \quad (\frac{u_{p}+u_{q_{2}}}{2}, \frac{v_{p}+v_{q_{2}}}{2}) * (-1, \ 0) + \\ 
                            \quad (\frac{u_{p}+u_{q_{3}}}{2}, \frac{v_{p}+v_{q_{3}}}{2}) * (\ 0, -1) + \\ 
                          = \frac{u_{q_{0}} - u_{q_{2}}}{2} + \frac{v_{q_{1}} - v_{q_{3}}}{2} \\ 
                          := \frac{\partial u}{\partial x} + \frac{\partial v}{\partial y}$$

从这个视角看公式(7)，其就是公式(4)的展开形式。此时方程的形式就十分明确了，代码会在最后给出。

# [无缝克隆](#无缝克隆)

对于向量场 $\mathbf{v}$ 的选择，简单的可以使用图像梯度，即:  

$$\mathbf{v} = \nabla g  \quad (9)$$

$$\Delta f=\Delta g \ \mathrm{over} \ \Omega, \ \mathrm{with} \ f|_{\partial \Omega} = f^{\star}|_{\partial \Omega} \quad(10)$$

或混合梯度

$$\mathrm{for} \ \mathrm{all} \ \mathbf{x}\in\Omega, \ \mathbf{v}(\mathbf{x}) = \left \{ \begin{array}{ll} \nabla f^{\star}(\mathbf{x}) & \textrm{if} \ |\nabla f^{\star}(x)|>|\nabla g(\mathbf{x})|, \\ \nabla g(\mathbf{x}) & \textrm{otherwise} \end{array} \right.   \quad (12)$$

与此对应

$$v_{pq} = \left \{ \begin{array}{ll} f^{\star}_{p} -f^{\star}_{q} & \textrm{if} \ | f^{\star}_{p} -f^{\star}_{q}|>| g_{p} - g_{q}| \\ g_{p}-g_{q} & \textrm{otherwise} \end{array} \right. \quad (13)$$

下图可以十分明晰的看出各种区别。

![]({{ site.baseurl }}/images/poissionImageEdit-002.png)

# [选区编辑](#选区编辑)

## [纹理拍平](#纹理拍平)

为了保留所选区域的显著特征，引导向量场可以设置为下面形式：

$$\mathrm{for} \ \mathrm{all} \ \mathbf{x}\in\Omega, \ \mathbf{v}(\mathbf{x}) = M(\mathbf{x})\nabla f^{\star}(\mathbf{x})  \quad (14)$$

其中 $M$ 时一个二元mask, 为了保留兴趣区域，当用边缘检测作为 $M$ 时，公式(7)中的 $v_{pq}$ 就是下面形式： 

$$v_{pq}=\left \{ \begin{array}{ll} f_{p}-f{q} & \textrm{if an edge lies between p and q} \\ 0 & \textrm{otherwise} \end{array}\right.  \quad (15)$$   

效果如下图：   
![]({{ site.baseurl }}/images/poissionImageEdit-003.png)

## [局部光照调整](#局部光照调整)

引导向量场可以设置如下：   

$$\mathbf{v}= \mathbf{\alpha}^{\beta} |\nabla f^{\star}|^{-\beta}\nabla f^{\star}  \quad (16)$$

其中 $\mathbf{\alpha}$ 是 $\Omega$ 上 $f^{\star}$ 的平均梯度场的0.2倍。      
其中 $\beta=0.2$    


效果如下图：    
![]({{ site.baseurl }}/images/poissionImageEdit-004.png)

## [局部色彩调整](#局部色彩调整)

1. 原始彩色图像 $g$ .    
2. 目标图像 $f^{\star}$ 是从 $g$ 获取对比度信息.    
3. 选择区域 $\Omega$ 包含目标物体。    
4. 公式(10)求解。   

（这段不是很明确，再想想）

![]({{ site.baseurl }}/images/poissionImageEdit-005.png)

## [无缝平铺](#无缝平铺)

公式如下：  

$$f^{\star}_{north}=f^{\star}_{south}=0.5(g_{north}+g_{south})$$

这里 $g$ 就是原始图像。

![]({{ site.baseurl }}/images/poissionImageEdit-006.png)





