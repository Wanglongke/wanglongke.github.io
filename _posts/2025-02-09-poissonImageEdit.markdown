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

对于 $S$ 上的任一像素 $p$ 其四邻域(上下左右)表示为 $N_{p}$ 其邻域对表示为 $<p,q>,\ q\in N_{p}$ . 区域 $\Omega$ 的边界表示为 $\partial \Omega = \{p \in S \backslash \Omega : N_{p} \cap \Omega \ne \emptyset \}$ . 用 $f_{p}$ 表示 $f$ 位于 $p$ 处的值。    

问题点的离散形式如下：

$$\min_{f|_{\Omega}} \sum_{<p,q>,p\in\Omega} (f_{p}-f_{q}-v_{pq})^{2}, \ \mathrm{with} f_{q}=f^{\star}_{q}, \ \mathrm{for} \ \mathrm{all} \ q \in \partial \Omega \quad (6)$$

其中    

$$v_{pq}=\mathbf{v}(\frac{p+q}{2}) \cdot \vec{pq}$$ 

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

# [实现](#实现)

代码较长...


```cpp
#include <iostream>
#include <stdexcept>
#include <string>
#include <opencv2/opencv.hpp>
#include <Eigen/Core>
#include <Eigen/Dense>
#include <Eigen/Sparse>

#define THROW_OR_TERMINATE(exception, message) throw exception(message)

#define ASSERT(condition, exception, message) \
if (!(condition)) { THROW_OR_TERMINATE(exception, std::string(__FILE__) + "(" + std::string(__FUNCTION__) + "," + std::to_string(__LINE__) + "): " + message + "\n"); }

#define LogAssert(condition, message) \
ASSERT(condition, std::runtime_error, message)

// mask(x)==64, Neumann bundary condition
// mask(x)==128, Dirichlet boundary condition
// mask(x)==255, interior region, equation(10)
// mask(x)==254, interior region, equation(12)
// mask(x)==253，texture flatten
// mask(x)==252, illumination changes

bool PoissonImageEdit(const cv::Mat& src, 
    const cv::Mat& mask, 
    const cv::Point &onDestOffset, 
    cv::Mat& dest)
{
    using vec3d = Eigen::Vector3d;
    using vec2d = Eigen::Vector2d;

    LogAssert(src.cols == mask.cols && (mask.cols + onDestOffset.x) <= dest.cols, "image size error");
    LogAssert(src.rows == mask.rows && (mask.rows + onDestOffset.y) <= dest.rows, "image size error");
    const int32_t w = src.cols;
    const int32_t h = src.rows;
    cv::Mat indices = cv::Mat::zeros(h, w, CV_32SC1) - 1;
    int32_t index = 0;
    for (int32_t iy = 0; iy < h; ++iy)
    {
        for (int32_t ix = 0; ix < w; ++ix)
        {
            if (!mask.at<uint8_t>(iy, ix)) { continue; }
            indices.at<int32_t>(iy, ix) = index;
            ++index;
        }
    }
    const int32_t nnz = index;
    if (nnz == 0) {
        return true;
    }

    const std::vector<cv::Point> coordDiff{
        cv::Point(-1, 0), cv::Point(1, 0),
        cv::Point(0, -1), cv::Point(0, 1)
    };

    auto lapFun = [](int32_t ix, int32_t iy, const cv::Mat& image)->vec3d
    {
        const cv::Vec3b& c01 = image.at<cv::Vec3b>(iy - 1, ix);
        const cv::Vec3b& c10 = image.at<cv::Vec3b>(iy, ix - 1);
        const cv::Vec3b& c11 = image.at<cv::Vec3b>(iy, ix);
        const cv::Vec3b& c12 = image.at<cv::Vec3b>(iy, ix + 1);
        const cv::Vec3b& c21 = image.at<cv::Vec3b>(iy + 1, ix);
        vec3d ret;
        for (int32_t ii = 0; ii < 3; ++ii)
        {
            ret[ii] = c01[ii] / 255.0 +
                c10[ii] / 255.0 -
                4.0 * c11[ii] / 255.0 +
                c12[ii] / 255.0 +
                c21[ii] / 255.0;
        }
        return ret;
    };

    auto gradFun = [](int32_t ix, int32_t iy, int32_t jx, int32_t jy, const cv::Mat& image)->vec3d
    {
        const cv::Vec3b& ci = image.at<cv::Vec3b>(iy, ix);
        const cv::Vec3b& cj = image.at<cv::Vec3b>(jy, jx);
        vec3d ret;
        for (int32_t ii = 0; ii < 3; ++ii)
        {
            ret[ii] = cj[ii] / 255.0 - ci[ii] / 255.0;
        }
        return ret;
    };

    bool initalMixGradientGuidedField = false;
    cv::Mat mixGradientGuidedFieldDiv;
    auto initMixFun = [&]() {
        mixGradientGuidedFieldDiv = cv::Mat(h, w, CV_32FC3, cv::Scalar(0, 0, 0));
        cv::Mat vx = cv::Mat(h, w, CV_32FC3, cv::Scalar(0, 0, 0));
        cv::Mat vy = vx.clone();
        for (int32_t iy = 0; iy < h; ++iy)
        {
            for (int32_t ix = 0; ix < w; ++ix)
            {
                if (!mask.at<uint8_t>(iy, ix)) { continue; }

                vec3d g_s_y = 0.5 * gradFun(ix, iy - 1, ix, iy + 1, src);
                vec3d g_s_x = 0.5 * gradFun(ix - 1, iy, ix + 1, iy, src);
                vec3d g_d_y = 0.5 * gradFun(ix + onDestOffset.x, iy - 1 + onDestOffset.y, ix + onDestOffset.x, iy + 1 + onDestOffset.y, dest);
                vec3d g_d_x = 0.5 * gradFun(ix - 1 + onDestOffset.x, iy + onDestOffset.y, ix + 1 + onDestOffset.x, iy + onDestOffset.y, dest);
                cv::Vec3f &vx_ = vx.at<cv::Vec3f>(iy, ix);
                cv::Vec3f &vy_ = vy.at<cv::Vec3f>(iy, ix);
                for (int ii = 0; ii < 3; ++ii)
                {
                    vec2d g_s(g_s_x[ii], g_s_y[ii]);
                    vec2d g_d(g_d_x[ii], g_d_y[ii]);
                    if (g_s.norm() > g_d.norm())
                    {
                        vx_[ii] = static_cast<float>(g_s[0]);
                        vy_[ii] = static_cast<float>(g_s[1]);
                    }
                    else
                    {
                        vx_[ii] = static_cast<float>(g_d[0]);
                        vy_[ii] = static_cast<float>(g_d[1]);
                    }
                }
            }
        }
    
        for (int32_t iy = 0; iy < h; ++iy)
        {
            for (int32_t ix = 0; ix < w; ++ix)
            {
                if (!mask.at<uint8_t>(iy, ix)) { continue; }
                cv::Vec3f& div = mixGradientGuidedFieldDiv.at<cv::Vec3f>(iy, ix);
                cv::Vec3f vxx = 0.5f * (vx.at<cv::Vec3f>(iy, ix + 1) - vx.at<cv::Vec3f>(iy, ix - 1));
                cv::Vec3f vyy = 0.5f * (vy.at<cv::Vec3f>(iy + 1, ix) - vy.at<cv::Vec3f>(iy - 1, ix));
                div = vxx + vyy;
            }
        }
    };

    bool initalEdgeDetect = false;
    cv::Mat edgeDetectDiv;
    auto initEdgeFun = [&]() {
        cv::Mat grayImage;
        cv::cvtColor(src, grayImage, cv::COLOR_BGR2GRAY);
        cv::blur(grayImage, grayImage, cv::Size(3, 3));
        cv::Mat edgeDetect;
        cv::Canny(grayImage, edgeDetect, 150, 100, 3);
        edgeDetectDiv = cv::Mat(h, w, CV_32FC3, cv::Scalar(0, 0, 0));
        cv::Mat vx = cv::Mat(h, w, CV_32FC3, cv::Scalar(0, 0, 0));
        cv::Mat vy = vx.clone();
        for (int32_t iy = 0; iy < h; ++iy)
        {
            for (int32_t ix = 0; ix < w; ++ix)
            {
                if (!mask.at<uint8_t>(iy, ix)) { continue; }
                uint8_t cur = edgeDetect.at<uint8_t>(iy, ix);
                bool onEdge = false;
                for (const cv::Point& dxy : coordDiff)
                {
                    int32_t jx = ix + dxy.x;
                    int32_t jy = iy + dxy.y;
                    if (jx < 0 || jx >= w || jy < 0 || jy >= h) {
                        continue;
                    }
                    if (edgeDetect.at<uint8_t>(jy, jx) != cur) {
                        onEdge = true;
                        break;
                    }
                }
                if (onEdge) {
                    vec3d g_s_y = 0.5 * gradFun(ix, iy - 1, ix, iy + 1, src);
                    vec3d g_s_x = 0.5 * gradFun(ix - 1, iy, ix + 1, iy, src);
                    vx.at<cv::Vec3f>(iy, ix) = cv::Vec3f(static_cast<float>(g_s_x[0]), 
                        static_cast<float>(g_s_x[1]), 
                        static_cast<float>(g_s_x[2]));
                    vy.at<cv::Vec3f>(iy, ix) = cv::Vec3f(static_cast<float>(g_s_y[0]), 
                        static_cast<float>(g_s_y[1]), 
                        static_cast<float>(g_s_y[2]));
                }
            }
        }
        for (int32_t iy = 0; iy < h; ++iy)
        {
            for (int32_t ix = 0; ix < w; ++ix)
            {
                if (!mask.at<uint8_t>(iy, ix)) { continue; }
                cv::Vec3f& div = edgeDetectDiv.at<cv::Vec3f>(iy, ix);
                cv::Vec3f vxx = 0.5f * (vx.at<cv::Vec3f>(iy, ix + 1) - vx.at<cv::Vec3f>(iy, ix - 1));
                cv::Vec3f vyy = 0.5f * (vy.at<cv::Vec3f>(iy + 1, ix) - vy.at<cv::Vec3f>(iy - 1, ix));
                div = vxx + vyy;
            }
        }
    };

    bool initalIllumination = false;
    cv::Mat illuminationDiv;
    auto initIlluminationFun = [&]() {
        illuminationDiv = cv::Mat(h, w, CV_32FC3, cv::Scalar(0, 0, 0));
        cv::Mat vx = cv::Mat(h, w, CV_32FC3, cv::Scalar(0, 0, 0));
        cv::Mat vy = vx.clone();

        vec3d sum_ng(0.0, 0.0, 0.0);
        int32_t count_vx_vy = 0;
        for (int32_t iy = 0; iy < h; ++iy)
        {
            for (int32_t ix = 0; ix < w; ++ix)
            {
                if (!mask.at<uint8_t>(iy, ix)) { continue; }
                vec3d g_s_y = 0.5 * gradFun(ix, iy - 1, ix, iy + 1, src);
                vec3d g_s_x = 0.5 * gradFun(ix - 1, iy, ix + 1, iy, src);
                
                vx.at<cv::Vec3f>(iy, ix) = cv::Vec3f(static_cast<float>(g_s_x[0]),
                    static_cast<float>(g_s_x[1]),
                    static_cast<float>(g_s_x[2]));
                vy.at<cv::Vec3f>(iy, ix) = cv::Vec3f(static_cast<float>(g_s_y[0]),
                    static_cast<float>(g_s_y[1]),
                    static_cast<float>(g_s_y[2]));
                count_vx_vy++;
                sum_ng[0] += vec2d(g_s_x[0], g_s_y[0]).norm();
                sum_ng[1] += vec2d(g_s_x[1], g_s_y[1]).norm();
                sum_ng[2] += vec2d(g_s_x[2], g_s_y[2]).norm();
            }
        }
        vec3d avg_ng = count_vx_vy > 0 ? sum_ng / (double)count_vx_vy : sum_ng;
        
        const float beta = 0.2f;
        cv::Vec3f alpha;
        alpha[0] = std::pow((float)avg_ng[0] * 0.2f, beta);
        alpha[1] = std::pow((float)avg_ng[1] * 0.2f, beta);
        alpha[2] = std::pow((float)avg_ng[2] * 0.2f, beta);
        
        for (int32_t iy = 0; iy < h; ++iy)
        {
            for (int32_t ix = 0; ix < w; ++ix)
            {
                if (!mask.at<uint8_t>(iy, ix)) { continue; }
                cv::Vec3f& vxi = vx.at<cv::Vec3f>(iy, ix);
                cv::Vec3f& vyi = vy.at<cv::Vec3f>(iy, ix);
                cv::Vec3f norm;
                norm[0] = std::sqrt(vxi[0] * vxi[0] + vyi[0] * vyi[0]) + 0.0001f;
                norm[1] = std::sqrt(vxi[1] * vxi[1] + vyi[1] * vyi[1]) + 0.0001f;
                norm[2] = std::sqrt(vxi[2] * vxi[2] + vyi[2] * vyi[2]) + 0.0001f;

                vxi[0] = std::pow(norm[0], -beta) * vxi[0] * alpha[0];
                vxi[1] = std::pow(norm[1], -beta) * vxi[1] * alpha[1];
                vxi[2] = std::pow(norm[2], -beta) * vxi[2] * alpha[2];
                vyi[0] = std::pow(norm[0], -beta) * vyi[0] * alpha[0];
                vyi[1] = std::pow(norm[1], -beta) * vyi[1] * alpha[1];
                vyi[2] = std::pow(norm[2], -beta) * vyi[2] * alpha[2];
            }
        }

        for (int32_t iy = 0; iy < h; ++iy)
        {
            for (int32_t ix = 0; ix < w; ++ix)
            {
                if (!mask.at<uint8_t>(iy, ix)) { continue; }
                cv::Vec3f& div = illuminationDiv.at<cv::Vec3f>(iy, ix);
                cv::Vec3f vxx = 0.5f * (vx.at<cv::Vec3f>(iy, ix + 1) - vx.at<cv::Vec3f>(iy, ix - 1));
                cv::Vec3f vyy = 0.5f * (vy.at<cv::Vec3f>(iy + 1, ix) - vy.at<cv::Vec3f>(iy - 1, ix));
                div = vxx + vyy;
            }
        }
    };

    std::vector<vec3d> coefficients_b;
    coefficients_b.resize(nnz);
    std::vector<Eigen::Triplet<double>> coefficients_A;
    coefficients_A.reserve(nnz);
    for (int32_t iy = 0; iy < h; ++iy)
    {
        for (int32_t ix = 0; ix < w; ++ix)
        {
            uint8_t im = mask.at<uint8_t>(iy, ix);
            if (!im) { continue; }
            const int32_t row = indices.at<int32_t>(iy, ix);
            if (im == 64)    // Neumann bundary condition
            {
                vec3d total_gs = vec3d(0.0, 0.0, 0.0);
                int32_t countNbr = 0;
                if (iy-1>=0 && mask.at<uint8_t>(iy - 1, ix)) {
                    int32_t i01 = indices.at<int32_t>(iy - 1, ix);
                    coefficients_A.push_back({ row, row, -1.0 });
                    coefficients_A.push_back({ row, i01, 1.0 });
                    vec3d g_s = gradFun(ix, iy, ix, iy - 1, src);
                    total_gs += g_s;
                    ++countNbr;
                }

                if (ix-1>=0 && mask.at<uint8_t>(iy, ix - 1)) {
                    int32_t i10 = indices.at<int32_t>(iy, ix - 1);
                    coefficients_A.push_back({ row, row, -1.0 });
                    coefficients_A.push_back({ row, i10, 1.0 });
                    vec3d g_s = gradFun(ix, iy, ix - 1, iy, src);
                    total_gs += g_s;
                    ++countNbr;
                }

                if (ix+1<w && mask.at<uint8_t>(iy, ix + 1)) {
                    int32_t i12 = indices.at<int32_t>(iy, ix + 1);
                    coefficients_A.push_back({ row, row, -1.0 });
                    coefficients_A.push_back({ row, i12, 1.0 });
                    vec3d g_s = gradFun(ix, iy, ix + 1, iy, src);
                    total_gs += g_s;
                    ++countNbr;
                }

                if (iy+1<h && mask.at<uint8_t>(iy + 1, ix)) {
                    int32_t i21 = indices.at<int32_t>(iy + 1, ix);
                    coefficients_A.push_back({ row, row, -1.0 });
                    coefficients_A.push_back({ row, i21, 1.0 });
                    vec3d g_s = gradFun(ix, iy, ix, iy + 1, src);
                    total_gs += g_s;
                    ++countNbr;
                }
                // LogAssert(countNbr!=0, "failed set mat");
                if (countNbr == 0)
                {
                    Eigen::Triplet<double> ti(row, row, 1.0);
                    coefficients_A.push_back(ti);
                    const cv::Vec3b& bgr = src.at<cv::Vec3b>(iy, ix);
                    coefficients_b[row][0] = bgr[0] / 255.0;
                    coefficients_b[row][1] = bgr[1] / 255.0;
                    coefficients_b[row][2] = bgr[2] / 255.0;
                }
                else
                {
                    coefficients_b[row] = total_gs;
                }
            }
            else if (im == 128)    // Dirichlet boundary condition
            {
                Eigen::Triplet<double> ti(row, row, 1.0);
                coefficients_A.push_back(ti);
                const cv::Vec3b& bgr = dest.at<cv::Vec3b>(iy + onDestOffset.y, ix + onDestOffset.x);
                coefficients_b[row][0] = bgr[0] / 255.0;
                coefficients_b[row][1] = bgr[1] / 255.0;
                coefficients_b[row][2] = bgr[2] / 255.0;
            }
            else if (im == 255)    // interior region, equation(10)
            {
                int32_t i01 = indices.at<int32_t>(iy - 1, ix);
                LogAssert(i01 != -1, "index error");
                int32_t i10 = indices.at<int32_t>(iy, ix - 1);
                LogAssert(i10 != -1, "index error");
                int32_t i11 = indices.at<int32_t>(iy, ix);
                LogAssert(i11 != -1, "index error");
                int32_t i12 = indices.at<int32_t>(iy, ix + 1);
                LogAssert(i12 != -1, "index error");
                int32_t i21 = indices.at<int32_t>(iy + 1, ix);
                LogAssert(i21 != -1, "index error");

                Eigen::Triplet<double> t01(row, i01, 1.0);
                Eigen::Triplet<double> t10(row, i10, 1.0);
                Eigen::Triplet<double> t11(row, i11, -4.0);
                Eigen::Triplet<double> t12(row, i12, 1.0);
                Eigen::Triplet<double> t21(row, i21, 1.0);
                coefficients_A.push_back(t01);
                coefficients_A.push_back(t10);
                coefficients_A.push_back(t11);
                coefficients_A.push_back(t12);
                coefficients_A.push_back(t21);
                vec3d l_s = lapFun(ix, iy, src);
                coefficients_b[row] = l_s;
            }
            else if (im == 254)    // interior region, equation(12)
            {
                int32_t i01 = indices.at<int32_t>(iy - 1, ix);
                LogAssert(i01 != -1, "index error");
                int32_t i10 = indices.at<int32_t>(iy, ix - 1);
                LogAssert(i10 != -1, "index error");
                int32_t i11 = indices.at<int32_t>(iy, ix);
                LogAssert(i11 != -1, "index error");
                int32_t i12 = indices.at<int32_t>(iy, ix + 1);
                LogAssert(i12 != -1, "index error");
                int32_t i21 = indices.at<int32_t>(iy + 1, ix);
                LogAssert(i21 != -1, "index error");

                Eigen::Triplet<double> t01(row, i01, 1.0);
                Eigen::Triplet<double> t10(row, i10, 1.0);
                Eigen::Triplet<double> t11(row, i11, -4.0);
                Eigen::Triplet<double> t12(row, i12, 1.0);
                Eigen::Triplet<double> t21(row, i21, 1.0);
                coefficients_A.push_back(t01);
                coefficients_A.push_back(t10);
                coefficients_A.push_back(t11);
                coefficients_A.push_back(t12);
                coefficients_A.push_back(t21);

                if (!initalMixGradientGuidedField)
                {
                    initalMixGradientGuidedField = true;
                    initMixFun();
                }
                const cv::Vec3f& div = mixGradientGuidedFieldDiv.at<cv::Vec3f>(iy, ix);
                coefficients_b[row] = vec3d(div[0], div[1], div[2]);
            }
            else if (im == 253)    // texture flatten
            {
                int32_t i01 = indices.at<int32_t>(iy - 1, ix);
                LogAssert(i01 != -1, "index error");
                int32_t i10 = indices.at<int32_t>(iy, ix - 1);
                LogAssert(i10 != -1, "index error");
                int32_t i11 = indices.at<int32_t>(iy, ix);
                LogAssert(i11 != -1, "index error");
                int32_t i12 = indices.at<int32_t>(iy, ix + 1);
                LogAssert(i12 != -1, "index error");
                int32_t i21 = indices.at<int32_t>(iy + 1, ix);
                LogAssert(i21 != -1, "index error");

                Eigen::Triplet<double> t01(row, i01, 1.0);
                Eigen::Triplet<double> t10(row, i10, 1.0);
                Eigen::Triplet<double> t11(row, i11, -4.0);
                Eigen::Triplet<double> t12(row, i12, 1.0);
                Eigen::Triplet<double> t21(row, i21, 1.0);
                coefficients_A.push_back(t01);
                coefficients_A.push_back(t10);
                coefficients_A.push_back(t11);
                coefficients_A.push_back(t12);
                coefficients_A.push_back(t21);
                if (!initalEdgeDetect)
                {
                    initalEdgeDetect = true;
                    initEdgeFun();
                }
                const cv::Vec3f& div = edgeDetectDiv.at<cv::Vec3f>(iy, ix);
                coefficients_b[row] = vec3d(div[0], div[1], div[2]);
            }
            else if (im == 252)  // local illumination changes
            {
                int32_t i01 = indices.at<int32_t>(iy - 1, ix);
                LogAssert(i01 != -1, "index error");
                int32_t i10 = indices.at<int32_t>(iy, ix - 1);
                LogAssert(i10 != -1, "index error");
                int32_t i11 = indices.at<int32_t>(iy, ix);
                LogAssert(i11 != -1, "index error");
                int32_t i12 = indices.at<int32_t>(iy, ix + 1);
                LogAssert(i12 != -1, "index error");
                int32_t i21 = indices.at<int32_t>(iy + 1, ix);
                LogAssert(i21 != -1, "index error");

                Eigen::Triplet<double> t01(row, i01, 1.0);
                Eigen::Triplet<double> t10(row, i10, 1.0);
                Eigen::Triplet<double> t11(row, i11, -4.0);
                Eigen::Triplet<double> t12(row, i12, 1.0);
                Eigen::Triplet<double> t21(row, i21, 1.0);
                coefficients_A.push_back(t01);
                coefficients_A.push_back(t10);
                coefficients_A.push_back(t11);
                coefficients_A.push_back(t12);
                coefficients_A.push_back(t21);
                if (!initalIllumination)
                {
                    initalIllumination = true;
                    initIlluminationFun();
                }
                const cv::Vec3f& div = illuminationDiv.at<cv::Vec3f>(iy, ix);
                coefficients_b[row] = vec3d(div[0], div[1], div[2]);
            }
        }
    }

    Eigen::SparseMatrix<double> A(nnz, nnz);
    A.setZero();
    A.setFromTriplets(coefficients_A.begin(), coefficients_A.end());
    Eigen::SparseLU<Eigen::SparseMatrix<double>> solver;
    solver.compute(A);
    if (solver.info() != Eigen::Success) {
        return false;
    }
    bool suc = true;
    Eigen::VectorXd xyz[3];
    for (int32_t ichannel = 0; ichannel < 3; ++ichannel)
    {
        Eigen::VectorXd b(nnz);
        for (int32_t i = 0; i < nnz; ++i)
        {
            b[i] = coefficients_b[i][ichannel];
        }
        Eigen::VectorXd x(nnz);
        x = solver.solve(b);
        if (solver.info() != Eigen::Success) {
            suc = false;
            break;
        }
        xyz[ichannel] = x;
    }
    if (!suc) {
        return false;
    }
    for (int32_t iy = 0; iy < h; ++iy)
    {
        for (int32_t ix = 0; ix < w; ++ix)
        {
            int32_t index = indices.at<int32_t>(iy, ix);
            if (index == -1) { continue; }
            cv::Vec3b& bgr = dest.at<cv::Vec3b>(iy + onDestOffset.y, ix + onDestOffset.x);
            bgr[0] = std::min(255, std::max(0, static_cast<int32_t>(xyz[0][index] * 255.0)));
            bgr[1] = std::min(255, std::max(0, static_cast<int32_t>(xyz[1][index] * 255.0)));
            bgr[2] = std::min(255, std::max(0, static_cast<int32_t>(xyz[2][index] * 255.0)));
        }
    }
    return true;
}

enum class EditType 
{
    SEAMLESS_CLONING=0,
    MIXED_SEAMLESS_CLONING=1,
    TEXTURE_FLATTENING=2,
    LOCAL_ILLUMINATION_CHANGES=3
};

void TestPoissionImageEdit(
    const std::string & srcImageFile,
    const std::string & maskImageFile,
    const std::string & destImageFile,
    const cv::Point &onDestOffset,
    const std::string & outputImageName,
    const EditType type
)
{
    cv::Mat srcImage = cv::imread(srcImageFile, cv::IMREAD_COLOR);
    LogAssert(!srcImage.empty(), "failed load source image");
    cv::Mat maskImage = cv::imread(maskImageFile, cv::IMREAD_GRAYSCALE);
    LogAssert(!maskImage.empty(), "failed load mask image");
    cv::Mat destImage = cv::imread(destImageFile, cv::IMREAD_COLOR);
    LogAssert(!destImage.empty(), "failed load dest image");
    
    const int w = maskImage.cols;
    const int h = maskImage.rows;
    for (int iy = 0; iy < h; ++iy)
    {
        for (int ix = 0; ix < w; ++ix)
        {
            uint8_t &val = maskImage.at<uint8_t>(iy, ix);
            if (val < 128) {
                val = 0;
            }
            else {
                val = 255;
            }
        }
    }
    const std::vector<cv::Point> coordDiff{
        cv::Point(-1, 0), cv::Point(1, 0),
        cv::Point(0, -1), cv::Point(0, 1)
    };
    for (int iy = 0; iy < h; ++iy)
    {
        for (int ix = 0; ix < w; ++ix)
        {
            if (!maskImage.at<uint8_t>(iy, ix)) {
                continue;
            }
            uint8_t onBoord = 0;
            uint8_t onBound = 0;
            for (const cv::Point& dxy : coordDiff) {
                int jx = ix + dxy.x;
                int jy = iy + dxy.y;
                if (jx < 0 || jx >= w || jy < 0 || jy >= h) {
                    onBoord = 1;
                    continue;
                }
                if (!maskImage.at<uint8_t>(jy, jx)) {
                    onBound = 1;
                }
            }
            if (onBoord) {
                maskImage.at<uint8_t>(iy, ix) = 64;
            }
            else if (onBound) {
                maskImage.at<uint8_t>(iy, ix) = 128;
            }
            else {
                if (type == EditType::SEAMLESS_CLONING) {
                    maskImage.at<uint8_t>(iy, ix) = 255;
                }
                else if (type == EditType::MIXED_SEAMLESS_CLONING) {
                    maskImage.at<uint8_t>(iy, ix) = 254;
                }
                else if (type == EditType::TEXTURE_FLATTENING)
                {
                    maskImage.at<uint8_t>(iy, ix) = 253;
                }
                else if (type == EditType::LOCAL_ILLUMINATION_CHANGES)
                {
                    maskImage.at<uint8_t>(iy, ix) = 252;
                }
            }
        }
    }
    cv::imwrite(outputImageName + "_mask.png", maskImage);
    std::cout << "prepare mask done" << std::endl;
    if (PoissonImageEdit(srcImage, maskImage, onDestOffset, destImage)) {
        std::cout << "sucess solve" << std::endl;
        cv::imwrite(outputImageName, destImage);
    }
    else {
        std::cout << "failed solve" << std::endl;
    }
}

void TestSeamlessTiling(const std::string &srcImageFile, 
                        const std::string &outputImageName,
                        const int32_t repeat_x=3,
                        const int32_t repeat_y=2)
{
    LogAssert(repeat_x > 0 && repeat_y > 0, "repeat times must greater than 0");
    cv::Mat srcImage = cv::imread(srcImageFile, cv::IMREAD_COLOR);
    LogAssert(!srcImage.empty(), "failed load source image");
    if (repeat_x == 1 && repeat_y == 1) {
        cv::imwrite(outputImageName, srcImage);
        return;
    }
    const int w = srcImage.cols;
    const int h = srcImage.rows;
    cv::Mat destImage;
    {
        std::vector<cv::Mat> himgs;
        himgs.resize(repeat_x, srcImage);
        cv::hconcat(himgs, destImage);
        std::vector<cv::Mat> vimgs;
        vimgs.resize(repeat_y, destImage);
        cv::vconcat(vimgs, destImage);
    }
    cv::Mat concatImage = destImage.clone();
    const int32_t dw = repeat_x * w;
    const int32_t dh = repeat_y * h;
    LogAssert(destImage.cols == dw && destImage.rows == dh, "image size error");
    cv::Mat maskImage = cv::Mat(dh, dw, CV_8UC1, cv::Scalar(255,255,255));
 
    for (int32_t irx = 0; irx < repeat_x; ++irx)
    {
        for (int32_t iry = 0; iry < repeat_y; ++iry)
        {
            for (int32_t iy = 0; iy < h; ++iy)
            {
                int32_t jx = irx * w;
                int32_t jy = iry * h + iy;
                const cv::Vec3b& bgr0 = srcImage.at<cv::Vec3b>(iy, 0);
                const cv::Vec3b& bgr1 = srcImage.at<cv::Vec3b>(iy, w-1);
                cv::Vec3b avg;
                avg[0] = (uint8_t)(((int32_t)bgr0[0] + (int32_t)bgr1[0]) / 2);
                avg[1] = (uint8_t)(((int32_t)bgr0[1] + (int32_t)bgr1[1]) / 2);
                avg[2] = (uint8_t)(((int32_t)bgr0[2] + (int32_t)bgr1[2]) / 2);
                int32_t jx_ = jx == 0 ? dw - 1 : jx - 1;
                destImage.at<cv::Vec3b>(jy, jx) = avg;
                destImage.at<cv::Vec3b>(jy, jx_) = avg;
                maskImage.at<uint8_t>(jy, jx) = 128;
                maskImage.at<uint8_t>(jy, jx_) = 128;
            }
        }
    }
    for (int32_t iry = 0; iry < repeat_y; ++iry)
    {
        for (int32_t irx = 0; irx < repeat_x; ++irx)
        {
            for (int32_t ix = 0; ix < w; ++ix)
            {
                int32_t jx = irx * w + ix;
                int32_t jy = iry * h;
                const cv::Vec3b& bgr0 = srcImage.at<cv::Vec3b>(0, ix);
                const cv::Vec3b& bgr1 = srcImage.at<cv::Vec3b>(h-1, ix);
                cv::Vec3b avg;
                avg[0] = (uint8_t)(((int32_t)bgr0[0] + (int32_t)bgr1[0]) / 2);
                avg[1] = (uint8_t)(((int32_t)bgr0[1] + (int32_t)bgr1[1]) / 2);
                avg[2] = (uint8_t)(((int32_t)bgr0[2] + (int32_t)bgr1[2]) / 2);
                int32_t jy_ = jy == 0 ? dh - 1 : jy - 1;
                destImage.at<cv::Vec3b>(jy, jx) = avg;
                destImage.at<cv::Vec3b>(jy_, jx) = avg;
                maskImage.at<uint8_t>(jy, jx) = 128;
                maskImage.at<uint8_t>(jy_, jx) = 128;
            }
        }
    }
     
    cv::imwrite(outputImageName + "_mask.png", maskImage);
    cv::imwrite(outputImageName + "_naive.png", destImage);
    std::cout << "prepare mask done" << std::endl;
    if (PoissonImageEdit(concatImage, maskImage, cv::Point(0, 0), destImage)) {
        std::cout << "sucess solve" << std::endl;
        cv::imwrite(outputImageName, destImage);
    }
    else {
        std::cout << "failed solve" << std::endl;
    }

}

int main(int argc, char* argv[])
{
	std::cout << "start..." << std::endl;
    const std::string curFile = std::string(__FILE__);
    const std::string curFileFolder = curFile.substr(0, curFile.find_last_of('\\'));
    // fuck windows path format
    
    std::cout << "case 1: " << std::endl; 
    {
        const std::string srcImageFile = curFileFolder + "\\data\\images\\1\\fg.jpg";
        const std::string maskImageFile = curFileFolder + "\\data\\images\\1\\mask.jpg";
        const std::string destImageFile = curFileFolder + "\\data\\images\\1\\bg.jpg";
        try {
            TestPoissionImageEdit(srcImageFile, maskImageFile, destImageFile, cv::Point(100, 100),
                curFileFolder + "\\data\\images\\1\\SEAMLESS_CLONING.png", EditType::SEAMLESS_CLONING);
        }
        catch (const std::exception& e) {
            std::cerr << "Error: " << e.what() << std::endl;
        }
        try {
            TestPoissionImageEdit(srcImageFile, maskImageFile, destImageFile, cv::Point(100, 100),
                curFileFolder + "\\data\\images\\1\\MIXED_SEAMLESS_CLONING.png", EditType::MIXED_SEAMLESS_CLONING);
        }
        catch (const std::exception& e) {
            std::cerr << "Error: " << e.what() << std::endl;
        }
        try {
            TestPoissionImageEdit(srcImageFile, maskImageFile, srcImageFile, cv::Point(0, 0),
                curFileFolder + "\\data\\images\\1\\TEXTURE_FLATTENING.png", EditType::TEXTURE_FLATTENING);
        }
        catch (const std::exception& e) {
            std::cerr << "Error: " << e.what() << std::endl;
        }
    }
    std::cout << "case 2: " << std::endl; 
    {
        const std::string srcImageFile = curFileFolder + "\\data\\images\\2\\fg.jpg";
        const std::string maskImageFile = curFileFolder + "\\data\\images\\2\\mask.jpg";
        const std::string destImageFile = curFileFolder + "\\data\\images\\2\\bg.jpg";
        try {
            TestPoissionImageEdit(srcImageFile, maskImageFile, destImageFile, cv::Point(150, 150),
                curFileFolder + "\\data\\images\\2\\SEAMLESS_CLONING.png", EditType::SEAMLESS_CLONING);
        }
        catch (const std::exception& e) {
            std::cerr << "Error: " << e.what() << std::endl;
        }
        try {
            TestPoissionImageEdit(srcImageFile, maskImageFile, destImageFile, cv::Point(150, 150),
                curFileFolder + "\\data\\images\\2\\MIXED_SEAMLESS_CLONING.png", EditType::MIXED_SEAMLESS_CLONING);
        }
        catch (const std::exception& e) {
            std::cerr << "Error: " << e.what() << std::endl;
        }
    }
    std::cout << "case 3: " << std::endl; 
    {
        const std::string srcImageFile = curFileFolder + "\\data\\images\\3\\fg.jpg";
        const std::string maskImageFile = curFileFolder + "\\data\\images\\3\\mask.jpg";
        const std::string destImageFile = curFileFolder + "\\data\\images\\3\\fg.jpg";
        try {
            TestPoissionImageEdit(srcImageFile, maskImageFile, destImageFile, cv::Point(0, 0),
                curFileFolder + "\\data\\images\\3\\LOCAL_ILLUMINATION_CHANGES.png", EditType::LOCAL_ILLUMINATION_CHANGES);
        }
        catch (const std::exception& e) {
            std::cerr << "Error: " << e.what() << std::endl;
        }
    }
    
    std::cout << "case 4: " << std::endl;
    {
        const std::string srcImageFile = curFileFolder + "\\data\\images\\4\\tile.jpg";
        try {
            TestSeamlessTiling(srcImageFile,
                curFileFolder + "\\data\\images\\4\\SEAMLESS_TILING.png");
        }
        catch (const std::exception& e) {
            std::cerr << "Error: " << e.what() << std::endl;
        }
    }

	std::cout << "finish..." << std::endl;
	return 0;
}


```



