---
layout: post
title:  倒数及平方根倒数
date:   2022-02-20 18:00:00 +0300
tags:   ComputerGraphics
description: 牛顿法求数字的“倒数”及“平方根倒数” 
---

倒数及平方根倒数的牛顿求解法，在书《Mathiematics for 3D Game Programming and Computer Graphics》139页。
# [目录](#目录)
1. [牛顿法](#牛顿法)
2. [倒数](#倒数)
3. [平方根倒数](#平方根倒数)
4. [很厉害的代码](#很厉害的代码)

# [牛顿法](#牛顿法)
![]({{ site.baseurl }}/images/inverseScalar-001.jpg)  
求函数 $f(x) = 0$ 的根。对于初始点 $x_{0}$ 根据其斜率的关系 $f^{'}(x_{0}) = \frac{(y - f(x_{0}))}{(x - x_{0})}$ 求其于x轴的相交，即 $y=0$ 处  $x$ 的值。可得到迭代的公式:  
$$ x_{i+1} = x_{i} - \frac{f(x_{i})}{f^{'}(x_{i})} $$    
另外可以证明牛顿迭代的收敛速度是二阶的。设 $r$ 是 $f(x) = 0$ 的根，对于第 $i$ 次迭代的结果 $x_{i}$ 有误差为:  
$$ \epsilon_{i} = x_{i} - r $$   
另外由迭代过程  
$$ x_{i+1} = x_{i} - \frac{f(x_{i})}{f^{'}(x_{i})} $$  
可知  
$$ \epsilon_{i+1} = x_{i+1} - r $$    
即    
$$ \epsilon_{i+1} = \epsilon_{i} - \frac{f(x_{i})}{f^{`}(x_{i})} $$    
令  
$$ g(x) = \frac{f(x)}{f^{'}(x)} $$  
则  
$$ g(x_{i}) = g(r + \epsilon_{i}) \approx g(r) + \epsilon_{i}g^{'}(r) + \frac{\epsilon_{i}^{2}}{2}g^{''}{r} $$    

$$ g^{'}(x) = 1 - \frac{f(x)f^{''}{x}}{[f^{'}(x)]^{2}}$$  

$$ g^{''}(x) = \frac{2f(x)f^{'}(x)[f^{''}(x)]^{2} - [f^{'}{x}]^{2}[f(x)f^{'''}(x)+f^{'}(x)f^{''}(x)]}{[f^{'}(x)]^{4}} $$   

由于 $f(r)=0$, 则可以计算    
$$ g(r) = 0 $$  
$$ g^{'}(r) = 1 $$  
$$ g^{''}(r) = - \frac{f^{''}(r)}{f^{'}(r)} $$  
则  
$$ g(x_{i}) \approx \epsilon_{i} - \frac{\epsilon_{i}^{2}}{2}\frac{f^{''}(r)}{f^{'}(r)} $$  
则  
$$ \epsilon_{i+1} = \frac{\epsilon_{i}^{2}}{2}\frac{f^{''}(r)}{f^{'}(r)} $$  
# [倒数](#倒数)

求 $r$ 的倒数 $\frac{1}{r}$ 就等价于求以下方程的根  
$$f(x)=x^{-1} - r$$  
当 $x=\frac{1}{r}$ 时 $f(x) = 0$.   
由牛顿迭代法可知迭代公式为  
$$x_{i+1} = x_{i} - \frac{x^{-1}_{i}-r}{-x^{-2}_{i}}$$   
$$x_{i+1} = x_{i}(2-rx_{i})$$  
通过迭代法可以进行求解，
```cpp
float inverseScalar(float s) {
	// fx = 1/x - r
	// search root, when x = 1/r, fx = 0
	float x0 = 1e-4f;
	float x1 = x0 * (2.f - s * x0);
	float eps = 1e-8f;
	int maxIter = 1000;
	int iter = 0;
	float error = 1.f;
	while (iter < maxIter && error > eps)
	{
		x1 = x0 * (2.f - s * x0);
		error = std::abs(x1 - x0);
		x0 = x1;
		iter++;
	}
	return x1;
}
```
# [平方根倒数](#平方根倒数)

求 $r$ 的平方根倒数 $\frac{1}{\sqrt{r}}$ 等价于求以下方程的根
$$f(x) = x^{-2} - r$$
牛顿迭代的公式为
$$x_{n+1} = x_{n} - \frac{x^{-2}_{n} - r}{-2x^{-3}_{n}}$$
$$x_{n+1} = \frac{1}{2}x_{n}(3-rx^{2}{n})$$
通过迭代法可进行求解
```cpp

float inverseSqrtScalar(float s) {
	// fx = 1/(x*x) - r
	// search root, when x = 1/sqrt(r), fx = 0
	float x0 = 1e-4f;
	float x1 = 0.5f * x0 * (3.f - s * x0 * x0);
	int maxIter = 1000;
	int iter = 0;
	float eps = 1e-8f;
	float error = 1.f;
	while (iter < maxIter && error > eps)
	{
		x1 = 0.5f * x0 * (3.f - s * x0 * x0);
		error = std::abs(x1 - x0);
		x0 = x1;
		iter++;
	}
	return x1;
}
```
当求解得到 $\frac{1}{\sqrt{r}}$，则可计算
$$\sqrt(r) = r(\frac{1}{\sqrt{r}})$$

--------------

测试
```cpp
int main() {

	float a = 5.f;
	float inva = 0.2f;
	float testInva = inverseScalar(a);
	std::cout.setf(std::ios::fixed);
	std::cout.precision(8);
	std::cout << a << ", inverse: " << testInva << std::endl;
	std::cout << "scalar error is " << std::abs(inva - testInva) << std::endl;
	float b = 2.f;
	float invsb = 1.f/std::sqrt(b);
	float testInvsb = inverseSqrtScalar(b);
	std::cout << b << ", sqrt inverse: " << testInvsb << std::endl;
	std::cout << "scalar error is " << std::abs(invsb - testInvsb) << std::endl;
	return 0;
}
```
结果如下
```
5.00000000, inverse: 0.19999999
scalar error is 0.00000001
2.00000000, sqrt inverse: 0.70710677
scalar error is 0.00000000
```
关于牛顿法的初值问题，这里暂且不进行讨论，明天吧。。。
# [很厉害的代码](#很厉害的代码)

雷神之锤三中有关于平方根倒数的很厉害的代码，有各种各样的详解，这里贴出来：
```cpp
float Q_rsqrt(float number)
{

	long i;
	float x2, y;
	const float threehalfs = 1.5F;

	x2 = number * 0.5F;
	y = number;
	i = *(long*)&y;                       // evil floating point bit level hacking
	i = 0x5f3759df - (i >> 1);                 // what the fuck?
	y = *(float*)&i;
	y = y * (threehalfs - (x2 * y * y));    // 1st iteration
	y = y * (threehalfs - (x2 * y * y));    // 2nd iteration, this can be removed

	return y;
}
```
结果如下，有点误差，但是它速度快呀。。。
```
2.00000000, sqrt inverse: 0.70710665
scalar error is 0.00000012
```
没有第二次迭代，结果如下，误差增加
```
2.00000000, sqrt inverse: 0.70693004
scalar error is 0.00017673
```