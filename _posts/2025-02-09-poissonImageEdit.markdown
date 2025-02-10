---
layout: post
title:  泊松图像编辑
date:   2025-01-15 15:30:00 +0300
tags:   ComputerVision
description:  泊松方程
---

# [简介](#简介)

参考论文：Pérez Patrick, Gangnet Michel, Blake Andrew. 2003. Poisson image editing. ACM Transactions on Graphics.    

# [引导插值](引导插值)

由于插值问题对于图像的各个通道的过程是独立的，一致的，这里用单通道做示例。

![]({{ site.baseurl }}/images/poissionImageEdit-001.png)

$S$ 是 $\mathbb{R}^2$ 上的封闭子集，就是图像空间。     
$\Omega$ 是 $S$ 上的一个封闭区域，其边界为 $\partial \Omega$.     

$f^{\star}$ 是$S-\partial \Omega$ 上的已知数值函数。   

$f$ 是 $\partial \Omega$ 上的未知数值函数。    

$\mathbf{v}$ 是 $\partial \Omega$ 上的向量场。  

最简单的插值就是"membrane"插值，其是下面问题的解：   

$$\min_{f} \int\int_{\Omega}\lvert\nabla f\rvert^{2} \mathrm{with} f\lvert_{\partial \Omega} = f^{\star}\lvert_{\partial \Omega}$$

# [无缝克隆](无缝克隆)


# [选区编辑](选区编辑)

