---
layout: post
title:  球极投影
date:   2024-08-01 15:30:00 +0300
tags:   ProjectiveGeometry
description:   射影几何
---

# [简介](#简介)

球极投影(stereographic projection)是一个映射，其将一个有限维欧式空间和比它高一维的欧式空间的单位球面去掉一个点形成的拓扑空间建立同胚。


![]({{ site.baseurl }}/images/stereographicProjection-001.jpg)  

球极投影是一种地图投影，$P$通过将球体表面上的点从球体的北极$N$投影到与南极$P^{'}$相切的平面上而获得。这种投影大圆被映射到园，斜航线变成对数螺旋线。

![]({{ site.baseurl }}/images/stereographicProjection-002.jpg)

球极投影具有非常简单的代数形式，可直接有三角形相似性得出。在上图中，让球的半径为$r$并且$z-$轴的位置如图所示，然后根据投影平面与$z-$轴的相对位置，可以得出不同的变换公式。


# [投影与逆投影](#投影与逆投影)

![]({{ site.baseurl }}/images/stereographicProjection-003.jpg)

这里以上图投影方式为例，演示投影过程。    

## [投影](#投影)

**N**和**S**分别为球的北极与南极，**P**为球面上一点，**Q**为其在**X-O-Y**平面上的球极投影点，$\hat{P}$为**P**的正交投影点。    

设  

$$P=(x_{p}, y_{p}, z_{p})$$

求解变换

$$Q=(x_{q}, y_{q}, z_{q}) = F(P)$$

这里根据三角形$P\hat{P}Q$与三角形$NOQ$相似可以得出：

$$\frac{\big\| Q-\hat{P} \big\|}{\big\| O-Q \big\|} = \frac{\big\| P-\hat{P} \big\|}{\big\| O-N \big\|}=\frac{z_{p}}{r}$$

又根据三角形$OP_{Y}\hat{P}$与三角形$OQ_{Y}Q$相似可以得出：    

$$\frac{\big\| O-P_{Y}\big\|}{\big\| O-Q_{Y} \big\|} = \frac{\big\|P_{Y}-\hat{P}\big\|}{\big\| Q_{Y}-Q\big\|}=\frac{\big\| O-\hat{P}\big\|}{\big\|O-Q\big\|}$$

因为$O\hat{P}Q$共线，所以：    

$$\frac{\big\| Q-\hat{P} \big\|}{\big\| O-Q \big\|}+\frac{\big\| O-\hat{P}\big\|}{\big\|O-Q\big\|}=1$$ 

所以：     

$$\frac{\big\| O-P_{Y}\big\|}{\big\| O-Q_{Y} \big\|}=\frac{y_{p}}{y_{q}}=\frac{x_{p}}{x_{q}}=1-\frac{z_{p}}{r}=\frac{r-z_{p}}{r}$$

所以：   

$$x_{q}=\frac{x_{p}*r}{r-z_{p}}$$   

$$y_{q}=\frac{y_{p}*r}{r-z_{p}}$$

## [逆投影](#逆投影)

已知平面$XOY$上任意一点**Q**求球面上一点**P**使得：  

$$Q=F(P)$$

即求变换$F^{-1}$使得：

$$F^{-1}(Q)=P$$

同理依据三角形相似可计算出：

$$\frac{x_{p}}{x_{q}}=\frac{y_p}{y_{q}}=\frac{r-z_{p}}{r}$$

又有：

$$\frac{\big\| O-\hat{P} \big\|}{\big\| O-Q \big\|}=\frac{x_{p}}{x_{q}}=\frac{y_p}{y_{q}}=\frac{r-z_{p}}{r}$$

令：

$$l=\big\| O-Q \big\|$$

所以：

$$\big\| O-\hat{P} \big\|= l *\frac{r-z_{p}}{r}$$

令：

$$a=\big\| O-\hat{P} \big\|$$

有：

$$a^{2} + z_{p}^{2}=r^{2}$$

且

$$\tan \angle NQO = \frac{r}{l} = \frac{z_{p}}{l-a}$$

所以得出:

$$a=\frac{2lr^{2}}{l^{2}+r^{2}}$$

所以：

$$x_{p} = \frac{a}{l}*x_{q}=\frac{2r^{2}}{l^{2}+r^{2}}*x_{q}$$

$$y_{p} = \frac{a}{l}*y_{q}=\frac{2r^{2}}{l^{2}+r^{2}}*y_{q}$$

$$z_{p} = \frac{r*(l-a)}{l}=\frac{r*(l^{2}-r^{2})}{l^{2}+r^{2}}$$

