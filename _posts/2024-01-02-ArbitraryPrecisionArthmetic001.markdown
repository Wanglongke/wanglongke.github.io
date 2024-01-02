---
layout: post
title:  ArbitraryPrecisionArthmetic-高精度计算
date:   2024-01-02 15:30:00 +0300
tags:   ArbitraryPrecisionArthmetic
description:  高精度计算
---

# [基本说明](#基本说明)

在解决几何问题的过程中使用浮点数计算并不总是符合算法预期的，我们所要求的计算精度的鲁棒性当前的浮点数会不满足，如IEEE 754-2008标准定义了32位和64位浮点数，即使已经有24位和53位的计算精度。下面介绍几个样例：

## [样例1](#样例1)

计算点$P$是否位于一个2D的凸多边形内，多边形有n个顶点$V_{i}$ 其中 $0\leq i<n$. 顶点为逆时针顺序.  

这要求所有的$\{P, V_{i}, V_{i+1}\}$ 也是逆时针顺序的。
``` cpp
template<typename Real>
int GetClassification(vec2<Real> const& p, int n, vec2<Real>const* v)
{
	int numPositive = 0;
	int numNegative = 0;
	int numZero = 0;
	for (int i0 = n - 1, i1 = 0; i1 < n; i0 = i1++)
	{
		vec2<Real> diff0 = p - v[i0];
		vec2<Real> diff1 = p - v[i1];
		Real dotPerp = diff0[0] * diff1[1] - diff0[1] * diff1[0];
		if (dotPerp > Real{ 0 })
		{
			++numPositive;
		}
		else if (dotPerp < Real{ 0 })
		{
			++numNegative;
		}
		else
		{
			++numZero;
		}
	}
	return (numZero == 0 ? (numPositive == n ? +1 : -1) : 0);
}


vec2<float> p(0.5, 0.5);
vec2<float> v[3];
v[0] = vec2<float>(-7.29045947e-013f, 6.29447341e-013f);
v[1] = vec2<float>(1.0f, 8.11583873e-013f);
v[2] = vec2<float>(9.37735566e-013f, 1.0f);
// the return value is 0, so p is determined to be on an edge of the 
// triangle, the function computes numPositive=2, numNegative=0 
// and numZero=1
int classifyUsingFloat = GetClassification<float>(p, 3, v);
// if rp nad rv[i] are ration representations of p and v[i] and you 
// have type ration that supports exact rational arithmetic, then
// the return value is +1, so p is actually strictly inside the triangle.
// the function computes numPositive=3, numNegative=0, and numero=0
int classifyUsingExact = GetClassification<Rational>(rp, 3, rv);
```

## [样例2](#样例2)
计算两条n维直线之间的距离。  
两个顶点$P_{0}$和$P_{1}$确定一条直线，参数表达为:  
$$P(s)=(1-s)P_{0} + sP_{1}, s\in\mathbf{R}$$.   
第二条直线由$Q_{0}$和$Q_{1}$确定, 参数表达为:  
$$Q(t)=(1-t)Q_{0} + tQ_{1}, t\in \mathbf{R}$$.  
两条直线上的点之间的距离的平方为:   
$$F(s,t)=\|P(s)-Q(t)\|^{2} = as^{2}-2bst+ct^{2}+2ds-et+f$$  
其中：  
$$a=\|P_{1}-P{0}\|^{2}, \qquad b=(P_{1}-P_{0})\cdot(Q_{1}-Q_{0}), \quad c=\|Q_{1}-Q_{0}\|^{2}, $$   
$$\qquad \qquad d=(P_{1}-P_{0})\cdot(P_{0}-Q_{0}), \qquad e=(Q_{1}-Q_{0})\cdot(P_{0}-Q_{0}), \quad f=\|P_{0}-Q_{0}\|^{2}$$  
如果$\quad ac-b^{2}=\|(P_{1}-P_{0})\times (Q_{1}-Q_{0})\|^{2}=0$ 则两条直线平行。  
如果$\quad ac-b^{2}>0$，则$F(s,t)$有全局最小值，当$\nabla F(s,t)=(0,0)$. 此时解$(\hat{s}, \hat{t})$分别代表两条直线距离最近是的参数。  
接下来求解：
$$\nabla F(s,t)=2(as-bt+d, -bs+ct-e)=(0,0)$$
即    
$$\left( \begin{array}{c} a & -b \\ -b & c\end{array} \right) \left( \begin{array}{c} s \\ t \end{array} \right) = \left( \begin{array}{c} -d \\ e \end{array} \right)$$
其解为： 
$$\left( \begin{array}{c} \hat{s} \\ \hat{t} \end{array} \right) = \frac{1}{ac-b^{2}} \left( \begin{array}{c} be-cd \\ ae-bd \end{array}\right)$$

```cpp
template<int32_t N, typename Real>
Real TestClosestPoint(const vec<N, Real>& p0, const vec<N, Real>& p1,
	                  const vec<N, Real>& q0, const vec<N, Real>& q1)
{
	vec<N, Real> p1mp0 = p1 - p0;
	vec<N, Real> q1mq0 = q1 - q0;
	vec<N, Real> p0mq0 = p0 - q0;
	Real a = p1mp0.dot(p1mp0);
	Real b = p1mp0.dot(q1mq0);
	Real c = q1mq0.dot(q1mq0);
	Real d = p1mp0.dot(p0mq0);
	Real e = q1mq0.dot(p0mq0);
	Real det = a * c - b * b;
	Real sNumber = b * e - c * d;
	Real tNumber = a * e - b * d;
	Real s = sNumber / det;
	Real t = tNumber / det;
	vec<N, Real> pclosest = (Real{ 1 } - s) * p0 + s * p1;
	vec<N, Real> qclosest = (Real{ 1 } - t) * q0 + t * q1;
	Real distance = (pclosest - qclosest).norm();
	return distance;
}

vec<3, double> p0(-1.0896217473782599, 9.7236145595088601e-007, 0.0);
vec<3, double> p1(0.91220578597858548, -9.4369829432107506e-007, 0.0);
vec<3, double> q0(-0.90010447502136237, 9.0671446351334441e-007, 0.0);
vec<3, double> q1(1.0730877178721130, -9.8185787633992740e-007, 0.0);
double dist = TestClosestPoint<3, double>(p0, p1, q0, q1);
// 0.43258687891076358
```

