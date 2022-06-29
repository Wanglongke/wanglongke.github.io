---
layout: post
title:  AVX-并行向量计算
date:   2022-02-10 10:30:00 +0300
tags:   ParallelProgramming
description: AVX(Advanced Vector Extension)简单使用
---

Intel有各种指令集，可在其[指令集页面](https://www.intel.com/content/www/us/en/docs/intrinsics-guide/index.html)进行检索。AVX指令集是Intel提供的并行矢量计算的指令集，是**C**风格的函数，提供对Intel的SIMD流扩展，高级矢量扩展和其他指令的访问，而无需编写汇编代码。

github上有相关的[示例代码](https://github.com/chen0031/AVX-AVX2-Example-Code#avx--avx2-intrinsics-example-code)可供直接阅读学习。


# [目录](#目录)

1. [指令名称简介](#指令名称简介)
2. [简单示例](#简单示例)
3. [扩展使用](#扩展使用)

# [指令名称简介](#指令名称简介)  

变量名称以 ```__m``` 开头(注意是两个下划线)；紧接着是变量位数 ```256```, ```128```； 其次是变量类型， ```float``` 类型没有额外表示， ```double``` 表示为```d```, ```int``` 表示为 ```i```. 如下：    

```__m256``` ： 表示 ```256```位 ```float``` 类型的数据格式，即 $256 / 32 = 8$ 个浮点数组成的向量。  
```__m256d```:  表示 ```256```位 ```double``` 类型的数据格式，即 $256 / 64 = 4$ 个双精度浮点数组成的向量。

```__m256i``` : 表示 ```256```位的整型数组成的向量。整型分为```32```位整型(int), ```64```位整型(int64_tx)(在```stdint.h```下可看各种整型的位数)


# [简单示例](#简单示例)  
x
# [扩展使用](#扩展使用)








