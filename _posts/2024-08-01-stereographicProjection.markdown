---
layout: post
title:  球极投影
date:   2024-08-01 15:30:00 +0300
tags:   ProjectiveGeometry
description:   射影几何
---

# [简介](#简介)

球极投影(stereographic projection)是一个映射，其将一个有限维欧式空间和比它高一维的欧式空间的单位球面去掉一个点形成的拓扑空间建立同胚。

# [详细](#详细)

![]({{ site.baseurl }}/images/stereographicProjection-001.jpg)  

球极投影是一种地图投影，$P$通过将球体表面上的点从球体的北极$N$投影到与南极$P^{'}$相切的平面上而获得。这种投影大圆被映射到园，斜航线变成对数螺旋线。

![]({{ site.baseurl }}/images/stereographicProjection-002.jpg)

球极投影具有非常简单的代数形式，可直接有三角形相似性得出。在上图中，让球的半径为$r$并且$z-$轴的位置如图所示，然后根据投影平面与$z-$轴的相对位置，可以得出不同的变换公式。

![]({{ site.baseurl }}/images/stereographicProjection-003.jpg)

半径为$R$的球体的变换方程为

$$x=k \cos \phi \sin(\lambda-\lambda_{0})$$

$$y=k[\cos \phi_{1} \sin \phi - \sin \phi_{1} \cos \phi \cos(\lambda - \lambda_{0})]$$

其中$\lambda_{0}$是中央经度，$\phi_{1}$是中央维度,

$$k=\frac{2R}{1+\sin \phi_{1} \sin \phi + \cos \phi_{1} \cos \phi \cos (\lambda - \lambda_{0})}$$

纬度与经度的逆公式如下：

$$\phi = \sin^{-1}(\cos c \sin \phi_{1} + \frac{y \sin c \cos \phi_{1}}{\rho})$$

$$\lambda = \lambda_{0} + \tan^{-1}(\frac{x\sin c}{\rho \cos \phi_{1} \cos c - y \sin \phi_{1} \sin c})$$

其中:

$$\rho = \sqrt{x^{2} + y^{2}}$$

$$c = 2\tan^{-1}(\frac{\rho}{2R})$$

# [样例](#样例)




