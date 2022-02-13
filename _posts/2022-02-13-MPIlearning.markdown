---
layout: post
title:  MPI(Massage Passing Interface)学习与训练
date:   2022-02-11 15:30:00 +0300
tags:   ParallelProgramming
description: 高性能计算之MPI并行程序设计
---

这里将记录MPI并行程序设计的学习过程。**MPI**全称**Massage Passing Interface**是多进程通讯协议，并不是一门语言，有不同组织的不同库，如微软的**MS-MPI**等。

# [目录](#目录)
1. [安装](#安装)
2. [并行程序设计简介](#并行程序设计简介)  
3. [MPI的基本用法](#MPI的基本用法)  
4. [未完待续](#未完待续)

# [安装](#安装)

## windows下环境配置
### 文件下载
Mircrosoft MPI(MS-MPI) 是微软的面向windows用户MPI编程的开发库。首先进入[Mircrosoft MPI](https://docs.microsoft.com/en-us/message-passing-interface/microsoft-mpi?redirectedfrom=MSDN)下载界面。  

![]({{ site.baseurl }}/images/mpi-001.png)  
*Minimalism*  
然后下载 ```msmoisetup.exe```, ```msmpisdk.msi```这两个文件并安装，默认会安装到Mircrosoft SDKs文件夹下。  
![]({{ site.baseurl }}/images/mpi-002.png)  
*Minimalism*  
1）在Visual Studio（我用的Visual Studio 2019）下新建空项目。  
2) 设置```Include Directories``` 和 ```Library Directories```, 其中注意选择```X64```的平台与文件夹。 
![]({{ site.baseurl }}/images/mpi-003.png)  
*Minimalism*    
3) 在链接器-->输入-->添加依赖库,```msmpi.lib```  
4) 设置预处理器定义, 追加 ```MPICH_SKIP_MPICXX```  
![]({{ site.baseurl }}/images/mpi-004.png)  
*Minimalism*  
5） 代码生成，选择```Multi-threaded Debug(/MTd)```  
![]({{ site.baseurl }}/images/mpi-005.png)  
*Minimalism*  
环境配置到此完成。新建```main.cpp```, 输入以下代码进行测试。
```cpp
#include <iostream>
#include <mpi.h>

int main(int argc, char* argv[]) {
    int id;
    int numprocess;
    MPI_Init(&argc, &argv);
    MPI_Comm_rank(MPI_COMM_WORLD, &id);
    MPI_Comm_size(MPI_COMM_WORLD, &numprocess);

    std::cout << "Process num: " << numprocess << ", id: " << id << std::endl;
    MPI_Finalize();
    return 0;
}
```
Build完成后在 ```*.exe``` 所在文件夹下执行测试。
```
mpiexec -np 4 .\learnOpenMPI.exe
```
其中```-np 4``` 是指用4个进程。 结果如下：
```
Process num: 4, id: 0
Process num: 4, id: 3
Process num: 4, id: 2
Process num: 4, id: 1

```
## linux下环境配置

# [并行程序设计简介](#目录)
并行计算机即能在同一时间内执行多条指令(或处理多个数据)的计算机，并行计算机是并行计算的物理载体。根据一个并行计算机能够同时执行的指令与处理数据的多少，可以把并行计算机分为**SIMD**(Single-Instruction Multiple-Data) 单指令多数据并行计算机和**MIMD**(Multiple-Instruction Multiple-Data)多指令多数据并行计算机。

# [MPI的基本用法](#目录)


# [未完待续](#目录)
