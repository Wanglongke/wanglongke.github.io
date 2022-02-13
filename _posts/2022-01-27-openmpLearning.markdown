---
layout: post
title:  openmp程序学习与训练(cpp+openmp)
date:   2022-01-27 17:00:00 +0300
tags:   学习与练习
description: 多线程并行工具的学习与练习                                                                                                                                                              
---

这里将会有一系列课程，用来记录我的编程练习。这一部分是关于并行程序设计课程中openmp的学习与练习。
# [目录](#目录)
1. [简介](#简介)
2. [基础语法](#基础语法)
3. [未完待续](#未完待续)

# [简介](#目录)
OpenMP(Open Multi-Processing) 属于共享内存编程模型的技术，程序员通过在源代码中加入指导性注释（Compiler Directive）、称为编译制导指令的专用的 ```#pragma``` 行来指明程序的并发属性。由此，编译器可以自动将程序并行化并在必要处加入同步互斥以及通信。  

由于OpenMP基于编译制导，具有简单、移植性好、可扩展性高以及支持增量并行化开发等优点，已经称为共享存储系统中的并行编程标准。  

支持语言，  ```c语言```, ```c++``` , ```Fortan```。 支持OpenMP的编译器有 ```GCC```, ```Omni```， ```OMPi```等编译器。  

可以说OpenMP制导指令将```C语言```扩展为一个并行语言，单OpenMP本身不是一种独立的并行语言，而是为多处理器上编写并行程序而设计的、指导共享内存多线程并行的编译制导指令和应用程序编程接口（API），可在C/C++和Fortan中应用。  

OpenMP的基本要素：  
1） 编译制导指令（Compiler Directive）  
2） 运行库（Runtime Library）  
3） 环境变量（Environment Variables）  
# [基础语法](#目录)

## 编译制导总览
在 ```C/C++``` 程序中，OpenMP的所有编译制导指令以 ```pragma omp``` 开始， 后面跟具体的功能指令或命令，具有以下形式：  
```cpp
#pragma omp 指令 [子句...]
```
指令或命令可以单独出现，子句必须出现在制导指令之后。制导指令和子句按照功能大体上分成四类:  
1）*并行域控制*  
2）*任务分担类*  
3）*同步控制类*  
4）*数据环境类*  

### [指令](#指令)  
(1) [parallel](#parallel): 用在一个结构块之前，表示这段代码将被多个线程执行；  
(2) [for](#for):用于for循环语句之前，表示将循环计算任务分配到多个线程中并行执行，以实现任务的分担，**必须由编程人员自己保证每次循环之间无数据相关性**；  
(3) [parallel for](#parallel-for): **parallel** 和 **for**指令的结合，用在for循环语句之前，表示for循环体的代码将被多个线程并行执行，它同时具有 *并行域的产生* 和 *任务分担* 两个功能；  
(4) [sections](#sections): 用在可被并行执行的代码段之前，用于实现多个结构快语句的任务分担，可并行执行的代码段各自用section指令标出（区分**sections**和**section**）；  
(5) [parallel sections](#parallel-sections): parallel和sections两个语句的结合，类似于parallel for；  
(6) [single](#single): 用在并行域内，表示一段只被单个线程执行的代码（注意于critical的区别）；  
(7) [critical](#critical): 用在一段代码临界区之前，保证每次只有一个OpenMP线程进入；  
(8) [flush](#flush): 保证各个OpenMP线程的数据影像的一致性；  
(9) [barrier](#barrier): 用于并行域内代码的线程同步，线程执行到barrier时要停下等待，直到所有线程执行到barrier时才继续往下执行；  
(10) [atomic](#atomic): 用于指定一个数据操作需要原子性的完成；  
(11) [master](#master): 用于指定一段代码由主线程执行；  
(12) [threadprivate](#threadprivate): 用于指定一个或多个变量是线程专有，后面会解释线程专有和线程私有的区别；  

### [子句](#子句)：  

(1) [private](#private): 指定一个或多个变量在每个线程中都有自己的私有副本；  
(2) [firstprivate](#firstprivate): 指定一个或多个变量在每个线程中都有自己的私有副本，并且私有变量要在进入并行域或任务分担时，继承主线程中的同名变量的值作为初值；  
(3) [lastprivate](#lastprivate): 使用来指定将线程中的一个或多个私有变量的值在并行处理结束后复制到主线程中的同名变量中，负责拷贝的线程是for或sections任务分担中的最后一个线程；  
(4) [reduction](#reduction): 用来指定一个或多个变量是私有的，并且在并行处理结束后这些变量要执行指定的归约运算，并将结果返回给主线程同名变量；  
(5) [nowait](#nowait): 指出并发线程可以忽略其他制导指令暗含的路障同步；  
(6) [num_threads](#num_threads): 指定并行域内的线程数目；  
(7) [schedule](#schedule): 指定for任务分担中的任务分配调度类型；  
(8) [shared](#shared): 指定一个或多个变量为多线程间的共享变量；  
(9) [ordered](#ordered): 用来指定for任务分担域内指定代码段需要按照串行顺序次序执行；  
(10) [copyprivate](#copyprivate): 配合single指令，将指定线程的专有变量广播到并行域内其他线程的同名变量；  
(11) [copyin](#copyin): 用来指定一个threadprivate类型的变量需要用主线程同名变量进行初始化；  
(12) [default](#default): 用来指定并行域内的变量使用方式，默认为shared；  

### [API函数](#API函数)：  
(1) [omp_in_parallel](#omp_in_parallel): 判断当前是否在并行域中；  
(2) [omp_get_thread_num](#omp_get_thread_num): 返回线程号；  
(3) [omp_set_num_threads](#omp_set_num_threads): 设置后续并行域中的线程个数；  
(4) [omp_get_num_threads](#omp_get_num_threads): 返回当前并行区域中的线程数；  
(5) [omp_get_max_threads](#omp_get_max_threads): 获取并行域可用的最大线程数目；  
(6) [omp_get_num_procs](#omp_get_num_procs): 返回系统中处理器个数；  
(7) [omp_get_dynamic](#omp_get_dynamic): 判断是否支持动态改变线程数目；  
(8) [omp_set_dynamic](#omp_set_dynamic): 启用或关闭线程数目的动态改变；  
(9) [omp_get_nested](#omp_get_nested): 判断系统是否支持并行嵌套；  
(10) [omp_set_nested](#omp_set_nested): 启用或关闭并行嵌套；  
(11) [omp_init(_nest)_lock](#omp_init(_nest)_lock): 初始化一个嵌套锁；  
(12) [omp_destory(_nest)_lock](#omp_destory(_nest)_lock): 销毁一个嵌套锁；  
(13) [omp_set(_nest)_lock](#omp_set(_nest)_lock): 嵌套加锁操作；  
(14) [omp_unset(_nest)_lock](#omp_unset(_nest)_lock): 嵌套解锁操作；  
(15) [omp_test(_nest)_lock](#omp_test(_nest)_lock): 非阻塞的嵌套加锁；  
(16) [omp_get_wtime](#omp_get_wtime): 获取wall time时间；  
(17) [omp_set_wtime](#omp_set_wtime): 设置wall time时间；  

### [环境变量](#环境变量)：  
(1) [OMP_SCHEDULE](#OMP_SCHEDULE): 用于for循环并行化后的调度，它的值就是训话调度的类型；  
(2) [OMP_NUM_THREADS](#OMP_NUM_THREADS): 用于设置并行域中的线程数；  
(3) [OMP_DYNAMIC](#OMP_DYNAMIC): 通过设定变量值，来确定是否允许动态设定并行域内的线程数；  
(4) [OMP_NESTED](#OMP_NESTED): 指出是否可以并行嵌套；

### [内部控制变量](#内部控制变量)  
**内部控制变量ICV(Internal Control Variable)** 用于表示系统的属性、能力和状态，可以通过OpenMP API函数访问也可以通过环境变量进行修改；

后面将详细介绍这些```指令```、```子句```、```API函数```的用法，这里仅仅列出供快速查询使用。

### 详细用法

##### [parallel](#指令)

##### [for](#指令)

##### [parallel for](#指令)

##### [sections](#指令)

# [未完待续](#目录)