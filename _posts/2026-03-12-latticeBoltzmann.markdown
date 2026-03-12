---
layout: post
title:  晶格玻尔兹曼仿真
data:   2026-03-11 15:30:00 +0300
tags:   ComputerGraphics
description:  LatticeBoltzmann
---

# [简介](#简介)

对此方法的归纳是在22年6月份做的，至今已接近4年，我这里尽量把之前归纳的内容还原到这里，此文档或许很长，有机会再跟几个文档系统性的介绍。         

# 基于格子Boltzmann方法的室内通风仿真及可视化

## 流体仿真方法介绍
流体在物理上是由大量粒子（约$10^{23}$量级）构成的离散系统，每个分子作无规则热运动，通过频繁的碰撞相互交换动量和能量。微观的流体粒子的运动在宏观上呈现出均匀性、连续性和确定性。流体系统的描述方法根据尺度的不同可分为微观分子模型、介观动理模型和宏观连续模型。基于格子Boltzmann方法的流体模型属于介观动理模型，用来描述流体微团的统计行为，不关心具体粒子的运动信息。

对于一个流体微团，具有密度$\rho$和体积$V_0$，则其质量为$m=\rho V_0$。考虑质量随时间的变化是由于当前体积下粒子的流入和流出，流体的质量不会凭空产生和消失，则可得到质量的连续性方程：    

$$
\frac{\partial m}{\partial t}=-\oint_{\partial V_0} \rho \boldsymbol{u} \cdot d\boldsymbol{A} \tag{1}
$$

$\boldsymbol{u}$为流体速度，对于封闭区域边界$\partial V_0$沿向外方向$d\boldsymbol{A}$的积分可以变为体积分：    

$$
\frac{\partial}{\partial t}\int_V \rho dV=-\int_V \nabla \cdot (\rho \boldsymbol{u}) dV \tag{2}
$$

最终得到偏微分方程：    

$$
\frac{\partial \rho}{\partial t}+\nabla \cdot (\rho \boldsymbol{u})=0 \tag{3}
$$

接下来考虑流体动量的变化，动量的改变由以下方面引起：     
1. 流入或流出流体的动量
2. 当前流体微团与外界的压强差
3. 外力作用    

$$
\frac{d}{dt}\int_V \rho \boldsymbol{u} dV=-\oint_{\partial V_0} \rho \boldsymbol{u}\boldsymbol{u} \cdot d\boldsymbol{A}-\oint_{\partial V_0} p d\boldsymbol{A}+\int_V \boldsymbol{F} dV \tag{4}
$$

其中    

$$
\boldsymbol{u}\boldsymbol{u}=\begin{pmatrix}
u_x u_x & u_x u_y & u_x u_z \\
u_y u_x & u_y u_y & u_y u_z \\
u_z u_x & u_z u_y & u_z u_z
\end{pmatrix}, \quad \boldsymbol{u}=(u_x,u_y,u_z)
$$

由此得到欧拉方程：    

$$
\frac{\partial (\rho \boldsymbol{u})}{\partial t}+\nabla \cdot (\rho \boldsymbol{u}\boldsymbol{u})=-\nabla p+\boldsymbol{F} \tag{5}
$$

考虑流体黏性带来的动量在流体间的转移与耗散，引入黏力后导出Navier-Stokes方程：    

$$
\frac{\partial \rho \boldsymbol{u}}{\partial t}=-\nabla \cdot (\rho \boldsymbol{u}\boldsymbol{u})-\nabla p+\eta \Delta \boldsymbol{u}+\boldsymbol{F} \tag{6}
$$

其中$\eta$为剪切黏性系数。当简单认为流体不可压缩，即$\rho$为常量，联合连续方程可得最终常见的不可压缩N-S方程组：     

$$
\frac{\partial \boldsymbol{u}}{\partial t}=-(\boldsymbol{u} \cdot \nabla)\boldsymbol{u}-\frac{1}{\rho}\nabla p+\nu \nabla^2 \boldsymbol{u}+\boldsymbol{F} \tag{7.1}
$$

$$
\nabla \cdot \boldsymbol{u}=0 \tag{7.2}
$$


其中$\nu=\frac{\eta}{\rho}$为运动粘度系数，$\nabla=(\frac{\partial}{\partial x},\frac{\partial}{\partial y},\frac{\partial}{\partial z})$，$\Delta=\nabla^2=(\frac{\partial^2}{\partial x^2},\frac{\partial^2}{\partial y^2},\frac{\partial^2}{\partial z^2})$。

### 数值求解流体方程组

求解方程组(7.1)、(7.2)最直接的方法是差分法对方程进行离散。首先对(7.1)的向量方程进行拆分得到方程组：   

$$
\frac{\partial u_x}{\partial t}+\frac{\partial u_x^2}{\partial x}+\frac{\partial u_x u_y}{\partial y}+\frac{\partial u_x u_z}{\partial z}=-\frac{\partial p}{\partial x}+F_x+\nu \left( \frac{\partial^2 u_x}{\partial x^2}+\frac{\partial^2 u_x}{\partial y^2}+\frac{\partial^2 u_x}{\partial z^2} \right) \tag{8.1}
$$  

$$
\frac{\partial u_y}{\partial t}+\frac{\partial u_y^2}{\partial y}+\frac{\partial u_y u_z}{\partial z}+\frac{\partial u_y u_x}{\partial x}=-\frac{\partial p}{\partial y}+F_y+\nu \left( \frac{\partial^2 u_y}{\partial x^2}+\frac{\partial^2 u_y}{\partial y^2}+\frac{\partial^2 u_y}{\partial z^2} \right) \tag{8.2}
$$

$$
\frac{\partial u_z}{\partial t}+\frac{\partial u_z^2}{\partial z}+\frac{\partial u_z u_x}{\partial x}+\frac{\partial u_z u_y}{\partial y}=-\frac{\partial p}{\partial z}+F_z+\nu \left( \frac{\partial^2 u_z}{\partial x^2}+\frac{\partial^2 u_z}{\partial y^2}+\frac{\partial^2 u_z}{\partial z^2} \right) \tag{8.3}
$$

对空间进行离散化，$x$方向离散为$x_0,\dots,x_i,\dots,x_{N_x}$，$y$方向为$y_0,\dots,y_j,\dots,y_{N_y}$，$z$方向为$z_0,\dots,z_k,\dots,z_{N_z}$，$x,y,z$三个方向的离散分别用下标$i,j,k$表示，即：   

$$
u_{i,j,k}=u(x_0+i \cdot \Delta x,y_0+j \cdot \Delta y,z_0+k \cdot \Delta z)
$$

其中$\Delta x,\Delta y,\Delta z$为空间步长。为使求解过程稳定，时间步长与空间步长需满足：    

$$
1>\max\left[ u_x \frac{\Delta t}{\Delta x},u_y \frac{\Delta t}{\Delta y},u_z \frac{\Delta t}{\Delta z} \right] \tag{9}
$$

对方程(8.1)中的偏导用差分进行逼近：   

$$
\begin{aligned}
\frac{u_x(i+1/2,j,k)^{n+1}-u_x(i+1/2,j,k)^n}{\Delta t}=&-\frac{u_x(i,j,k)^{2}-u_x(i+1,j,k)^{2}}{\Delta x}\\
&-\frac{(u_x u_y)(i+1/2,j+1/2,k)-(u_x u_y)(i+1/2,j-1/2,k)}{\Delta y}\\
&-\frac{(u_x u_z)(i+1/2,j,k+1/2)-(u_x u_z)(i+1/2,j,k-1/2)}{\Delta z}\\
&+\frac{p(i,j,k)-p(i+1,j,k)}{\Delta x}\\
&+\frac{\nu}{\Delta x^2}\left[ u_x(i+1/2,j+1,k)-2u_x(i+1/2,j,k)+u_x(i-1/2,j,k) \right]\\
&+\frac{\nu}{\Delta y^2}\left[ u_x(i+1/2,j+1,k)-2u_x(i+1/2,j,k)+u_x(i+1/2,j-1,k) \right]\\
&+\frac{\nu}{\Delta z^2}\left[ u_x(i+1/2,j,k+1)-2u_x(i+1/2,j,k)+u_x(i+1/2,j,k-1) \right]
\end{aligned} \tag{10}
$$


方程(8.2)(8.3)的离散方式同上。此时方程组有4个未知数$u_x,u_y,u_z,p$，其中压强参数由方程(7.2)求出。   

计算“丢失”质量：  

$$
\begin{aligned}
D(i,j,k)=&-\frac{u_x(i+1/2,j,k)-u_x(i-1/2,j,k)}{\Delta x}\\
&-\frac{u_y(i,j+1/2,k)-u_y(i,j-1/2,k)}{\Delta y}\\
&-\frac{u_z(i,j,k+1/2)-u_z(i,j,k-1/2)}{\Delta z}
\end{aligned} \tag{11}
$$

根据“丢失”质量计算压强差：  

$$
\Delta p=\beta D \tag{12}
$$

其中$\beta$计算如下：   

$$
\beta=\frac{\beta_0}{2}\left( \frac{\Delta t}{\Delta x^2}+\frac{\Delta t}{\Delta y^2}+\frac{\Delta t}{\Delta z^2} \right) \tag{13}
$$

$\beta_0 \in [1,2]$

速度场矫正：  

$$
u_x(i+1/2,j,k)=u_x(i+1/2,j,k)+\frac{\Delta t \Delta p}{\Delta x} \tag{14.1}
$$

$$
u_x(i-1/2,j,k)=u_x(i-1/2,j,k)-\frac{\Delta t \Delta p}{\Delta x} \tag{14.2}
$$

$u_y,u_z$方向同理。

更新压强场：   

$$
p(i,j,k)=p(i,j,k)+\Delta p \tag{15}
$$

**算法：数值求解N-S-E Eq.(7.1~7.2)**
1. 空间离散化确定$\Delta t,\Delta x,\Delta y,\Delta z$（Eq.(9)）
2. 初始化速度$\boldsymbol{u}$，压强$p$
3. while $t_i < t_{end}$
    1. 计算新的速度场$\boldsymbol{u}$（Eq.(10)）
    2. 计算“丢失”质量$D$（Eq.(11)）
    3. 计算压强差$\Delta p$（Eq.(12)(13)）
    4. 速度场矫正（Eq.(14.1)(14.2)）
    5. 更新压强$p$（Eq.(15)）
    6. 更新时间$t_i=t_i+\Delta t$
4. 求解结束

### Boltzmann方法仿真

从麦克斯韦的观点出发，任意时刻单个分子的速度和位置信息并不重要，分布函数才是描述分子效应的重要参数。流体分子碰撞遵循动量守恒，热平衡时分布函数各向同性，即Maxwell-Boltzmann分布：   

$$
f(x,|\boldsymbol{\xi}|,t)=\left( \frac{m}{2\pi k_B T} \right)^{\frac{3}{2}} e^{-\frac{m|\boldsymbol{\xi}|^2}{2k_B T}} \tag{16}
$$

其中$k_B=1.38×10^{-23} \ \text{J/K}$为玻尔兹曼常数。

对分布$f$关于时间$t$微分：   

$$
\frac{df}{dt}=\frac{\partial f}{\partial t}+\boldsymbol{\xi} \cdot \frac{\partial f}{\partial \boldsymbol{x}}+\frac{\boldsymbol{F}}{m} \cdot \frac{\partial f}{\partial \boldsymbol{\xi}} \tag{17.1}
$$

$$
\frac{d\boldsymbol{x}}{dt}=\boldsymbol{\xi} \tag{17.2}
$$

$$
\frac{d\boldsymbol{\xi}}{dt}=\boldsymbol{a}=\frac{\boldsymbol{F}}{m} \tag{17.3}
$$

得到Boltzmann方程：   

$$
\frac{\partial f}{\partial t}+\boldsymbol{\xi} \cdot \nabla f+\frac{\boldsymbol{F}}{m} \cdot \nabla_{\boldsymbol{\xi}} f=\Omega(f) \tag{18}
$$

$\Omega(f)$为碰撞算子，常用BGK碰撞算子：   

$$
\Omega(f)=-\omega(f-f^{\text{eq}})=-\frac{1}{\tau}(f-f^{\text{eq}}) \tag{19}
$$

$\tau$为松弛因子，$\omega=1/\tau$为碰撞频率。

LBM基本量为离散速度分布函数$f_i(\boldsymbol{x},t)$，宏观密度与动能：   

$$
\rho(\boldsymbol{x},t)=\sum_i f_i(\boldsymbol{x},t) \tag{21.1}
$$

$$
\rho \boldsymbol{u}(\boldsymbol{x},t)=\sum_i \boldsymbol{c}_i f_i(\boldsymbol{x},t) \tag{21.2}
$$

速度集记为$DdQq$，$d$为维度，$q$为速度数量，常用$D1Q3、D2Q9、D3Q15、D3Q19、D3Q27$。

离散Boltzmann方程（BGK）：   

$$
f_i(\boldsymbol{x}+\Delta \boldsymbol{x},t+\Delta t)=f_i(\boldsymbol{x},t)-\frac{1}{\tau}(f_i(\boldsymbol{x},t)-f_i^{\text{eq}}(\boldsymbol{x},t)) \tag{23}
$$

平衡分布函数：   

$$
f_i^{\text{eq}}(\boldsymbol{x},t)=w_i \rho \left( 1+\frac{\boldsymbol{u} \cdot \boldsymbol{c}_i}{c_s^2}+\frac{(\boldsymbol{u} \cdot \boldsymbol{c}_i)^2}{2c_s^4}-\frac{\boldsymbol{u} \cdot \boldsymbol{u}}{2c_s^2} \right) \tag{25}
$$

其中声速$c_s^2=1/3$。

**算法：LBM简单计算过程**
1. while $t_i < t_{end}$
    1. 通过$f_i$计算密度$\rho$和速度$\boldsymbol{u}$
    2. 通过$\rho,\boldsymbol{u}$计算平衡状态$f_i^{\text{eq}}$
    3. BGK碰撞算子求解下一时刻$f_i^*$
    4. 结果传播流动
    5. 更新时间
2. 计算结束

### 光滑粒子法仿真
是不是不用介绍呢？？？

## 基于格子Boltzmann方法的室内通风仿真
### 空间离散化
格子Boltzmann方法将空间离散为均匀格子，对房屋三角网格模型（含房屋与家具）进行离散处理：
1. 取房屋模型包围盒构建格子空间$N_x×N_y×N_z$，统一空间步长$\Delta x$
2. 对房屋、家具模型构建八叉树
3. 射线交叉检测区分室内/室外格子
4. 判断固体边界格子

**格子类型划分**   

| 格子类型 | 格子类型ID |
|----------|------------|
| 室外     | 0          |
| 室内空区域 | 1        |
| 固体边界 | 2          |

剔除内部无用格子，仅保留连通性最大的流体格子。

### 物理参数选取
依据**相似定理**，两不可压缩流体系统动态相似的条件：相同雷诺数+几何相似。
雷诺数：  

$$
Re=\frac{lU}{\nu} \tag{26}
$$

$l$为特征长度，$U$为特征速度，$\nu$为运动粘度。

物理量转换因子：  

$$
C_l=\frac{\Delta x}{\Delta x^*}=\Delta x \tag{28.1}
$$

$$
C_t=\frac{\Delta t}{\Delta t^*}=\Delta t \tag{28.2}
$$

$$
C_{\rho}=\frac{\rho}{\rho^*}=\rho \tag{28.3}
$$

粘度与松弛因子关系：   

$$
\nu=c_s^{*2}\left( \tau^*-\frac{1}{2} \right)\frac{\Delta x^2}{\Delta t} \tag{29}
$$

速度转换：   

$$
C_u=\frac{C_l}{C_t} \tag{30}
$$

常温空气粘度$\nu=1.58×10^{-5} \ \text{m}^2/\text{s}$，松弛因子：  

$$
\tau^*=\nu \cdot \frac{\Delta t}{\Delta x^2} \cdot \frac{1}{c_s^{*2}}+0.5 \tag{31}
$$

稳定性要求：速度远小于声速$c_s^*=1/\sqrt{3}$，松弛因子$\tau^*>0.55$。   

### 定义边界条件
入口设置**速度边界**，出口设置**恒压边界**。

### 仿真过程
仿真过程中保存结果，支持动态改变边界条件。

## 室内仿真结果的展示与分析
三维速度场可视化常用**粒子法(stream particle)**和**流线法(stream line)**，均基于Runge-Kutta(R-K)方法计算粒子运动路径：  

$$
\boldsymbol{p}_{i+1}=\boldsymbol{p}_i+\frac{\Delta t}{6}\left( \boldsymbol{v}_i+2\boldsymbol{v}_{i+1}^1+2\boldsymbol{v}_{i+1}^2+\boldsymbol{v}_{i+1}^3 \right) \tag{31}
$$

其中：   

$$
\boldsymbol{p}_{i+1}^k=\boldsymbol{p}_i+\frac{1}{2}\Delta t \boldsymbol{v}_{i+1}^{k-1} \tag{32.1}
$$

$$
\boldsymbol{v}_{i+1}^0=\boldsymbol{v}_i \tag{32.2}
$$

流线法将粒子运动路径连线，直观呈现室内气流分布。

![]({{ site.baseurl }}/images/lattice-001.png)

## 户型图上的仿真结果展示

![]({{ site.baseurl }}/images/lattice-002.png)

![]({{ site.baseurl }}/images/lattice-003.png)







