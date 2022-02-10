---
layout: post
title:  openmp程序学习与训练(cpp+openmp)
date:   2022-01-27 17:00:00 +0300
tags:   学习与练习
description: 多线程并行工具的学习与练习                                                                                                                                                              
---

这里将会有一系列课程，用来记录我的编程练习。这一部分是关于并行程序设计课程中openmp的学习与练习。
## 目录
1. [简介](#简介)
2. [基础语法](#基础语法)
3. [未完待续](#未完待续)

## [简介](#目录)
OpenMP(Open Multi-Processing) 属于共享内存编程模型的技术，程序员通过在源代码中加入指导性注释（Compiler Directive）、称为编译制导指令的专用的 ```#pragma``` 行来指明程序的并发属性。由此，编译器可以自动将程序并行化并在必要处加入同步互斥以及通信。  

由于OpenMP基于编译制导，具有简单、移植性好、可扩展性高以及支持增量并行化开发等优点，已经称为共享存储系统中的并行编程标准。  

支持语言，  ```c语言```, ```c++``` , ```Fortan```。 支持OpenMP的编译器有 ```GCC```, ```Omni```， ```OMPi```等编译器。  

可以说OpenMP制导指令将```C语言```扩展为一个并行语言，单OpenMP本身不是一种独立的并行语言，而是为多处理器上编写并行程序而设计的、指导共享内存多线程并行的编译制导指令和应用程序编程接口（API），可在C/C++和Fortan中应用。  

OpenMP的基本要素：  
1） 编译制导指令（Compiler Directive）  
2） 运行库（Runtime Library）  
3） 环境变量（Environment Variables）  
## [基础语法](#目录)

### 编译制导总览
在 ```C/C++``` 程序中，OpenMP的所有编译制导指令以 ```pragma omp``` 开始， 后面跟具体的功能指令或命令，具有以下形式：  
```cpp
#pragma omp 指令 [子句...]
```
指令或命令可以单独出现，子句必须出现在制导指令之后。制导指令和子句按照功能大体上分成四类:  
1）并行域控制  
2）任务分担类  
3）同步控制类  
4）数据环境类  

#### 指令  
(1) [parallel](#parallel)  
(2) [for](#for)  
(3) [parallel for](#parallel-for)  
(4) [sections](#sections)  
(5) [parallel sections](#parallel-sections)  
(6) [single](#single)  
(7) [critical](#critical)  
(8) [flush](#flush)  
(9) [barrier](#barrier)  
(10) [atomic](#atomic)  
(11) [master](#master)  
(12) [threadprivate](#threadprivate)  

#### 子句：  

(1) private:  
(2) firstprivate:  
(3) lastprivate:  
(4) reduction:  
(5) nowait:  
(6) num_threads:  
(7) schedule:  
(8) shared:  
(9) ordered:  
(10) copyprivate:  
(11) copyin:  
(12) default:  

```API函数```如下：  

```环境变量```如下：  


后面将详细介绍这些```指令```、```子句```、```API函数```的用法，这里仅仅列出供快速查询使用。

### 详细用法

#### [parallel](#指令)

#### [for](#指令)

#### [parallel for](#指令)

#### [sections](#指令)

## [未完待续](#目录)