---
layout: post
title:  基于硬件加速的多视几何优化
date:   2025-02-27 15:30:00 +0300
tags:   ComputerGraphics
description:  多视几何
---

# [简介](#简介)

参考文献：     


Vu Hoang-Hiep, Labatut Patrick, Pons Jean-Philippe, Keriven Renaud. High accuracy and visibility-consistent dense multiview stereo. IEEE Transactions on Pattern Analysis and Machine Intelligence. 2012.

# [输入](#输入)

多视图像：    

![]({{ site.baseurl }}/images/photoCMR-data-001.png)  


![]({{ site.baseurl }}/images/photoCMR-data-002.png)


# [硬件加速](#硬件加速)

采用OpenGL对运算过程进行加速。包括用于渲染的顶点、几何、面片着色器，通用的计算着色器，对多个计算过程进行加速。

# [效果](#效果)

初始模型：    

![]({{ site.baseurl }}/images/photoCMR-002.png)

优化后的模型：

![]({{ site.baseurl }}/images/photoCMR-001.png)



