---
layout: post
title:  STL-Stream
date:   2024-01-04 15:30:00 +0300
tags:   cpp
description:  cpp标准模板库的IO流
---

# [基本说明](#基本说明)

cpp标准模板库的IO流用来读入或写出。这里有文件流和数据流。本文介绍他们之间的继承关系。以及简单功能。

# [继承关系图](#继承关系图)

各个流之间继承关系如下    

<span class="mermaid">
flowchart TD  
    A[ios] --> B[istream];
    A[ios] --> C[ostream];
    B[istream] --> D[istringstream];
    B[istream] --> E[ifstream];
    B[istream] --> F[iostream];
    C[ostream] --> G[ostringstream];
    C[ostream] --> H[ofstream];
    C[ostream] --> F[iostream];
    F[iostream] --> I[fstream];
    F[iostream] --> J[stringstream]
</span>
<script src="https://cdn.jsdelivr.net/npm/mermaid/dist/mermaid.min.js"></script>

各个内存缓存继承关系如下    

<span class="mermaid">
flowchart TD  
    A[streambuf] --> B[stringbuf];
    A[streambuf] --> C[filebuf];
</span>
<script src="https://cdn.jsdelivr.net/npm/mermaid/dist/mermaid.min.js"></script>


