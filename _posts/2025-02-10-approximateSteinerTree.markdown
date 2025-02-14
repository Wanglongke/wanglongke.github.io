---
layout: post
title:  近似斯坦纳树
date:   2025-01-15 15:30:00 +0300
tags:   ComputerGraphics
description:  斯坦纳树
---

# [简介](#简介)

斯坦纳树问题是组合优化问题，与最小生成树相似，是最短网络的一种。      

**最小生成树**：在给定的图(graph)上找到最短的连通树，连通树包含图上所有顶点，不包含所有边，使得将连通树的边长累加在一起后达到最小值。     

**最小斯坦纳树**: 在给定的图(graph)上，给定一组点，在图上找到一组边能把所有的顶点连接在一起，并且这组边的边长累加在一起能达到最小值。     

这两个问题都是在图上找到一个边的子集，这个子集满足特定条件：   
1）最小生成树满足：边端点的集合等于图顶点的集合，边长和最小。      
2）最小斯坦纳树满足：边端点的集合包含给定点集，边长和最小。   

# [问题提出](#问题提出)

将三个村庄用总长为极小的道路连接起来。从数学上说，就是在平面内给定三个点 A、B、C 找出平面内第四个点 P，使得和数 a+b+c 为最短，这里 a、b、c 分别表示从 P 到 A、B、C 的距离。     

答案：如果三角形 $\textit{ABC}$ 的每个内角都小于 $120^{\circ}$，那么 P 就是使边 $\textit{AB}$、$\textit{BC}$、$\textit{AC}$ 对该点所张的角都是 $120^{\circ}$ 的点。如果三角形 $\textit{ABC}$ 的有一个角，例如 C 角，大于或等于 $120^{\circ}$，那么点 P 与顶点 C 重合。

# [近似斯坦纳树](#近似斯坦纳树)

参考论文：Seamster: Inconspicuous low-distortion texture seam layout     

![]({{ site.baseurl }}/images/steinerTree-001.jpg)

# [主要思路](#主要思路)

1. 给定点为种子点，广度优先搜索；     
2. 搜索队列为优先级队列，每次从最近的位置开始；     
3. 当多源的搜索碰撞，则记录路径；    


# [实现](#实现)

```cpp
#pragma once

#include <vector>
#include <unordered_map>
#include <map>
#include <stack>


template<typename NodeType, typename WeightType>
class SteinerTree
{

protected:
    using IdxType = int32_t;
    std::unordered_map<NodeType, IdxType> mNodeToIndex;
    std::vector<NodeType> mIndexToNode;
    std::vector<std::map<IdxType, WeightType>> mNbrMaps;

public:
    SteinerTree()=default;
    ~SteinerTree()=default;

    void Init(IdxType numNode)
    {
        mNbrMaps.reserve(numNode);
        mNodeToIndex.reserve(numNode);
        mIndexToNode.reserve(numNode);
    }

    void Clear()
    {
        mNbrMaps.clear();
        mNodeToIndex.clear();
        mIndexToNode.clear();
    }

    void UpdateEdge(const NodeType& i0, const NodeType &i1, const WeightType &w)
    {
        IdxType n0, n1;
        auto it0 = mNodeToIndex.find(i0);
        if(it0==mNodeToIndex.end())
        {
            n0 = mIndexToNode.size();
            if(n0==std::numeric_limits<IdxType>::max()){
                LogError("Node number boom!!! Please modify IdxType!!!");
            }
            mIndexToNode.push_back(i0);
            mNbrMaps.push_back(std::map<IdxType, WeightType>{});
            mNodeToIndex.insert({i0, n0});
        }
        else
        {
            n0 = it0->second;
        }
        auto it1 = mNodeToIndex.find(i1);
        if(it1==mNodeToIndex.end())
        {
            n1 = mIndexToNode.size();
            if(n1==std::numeric_limits<IdxType>::max()){
                LogError("Node number boom!!! Please modify IdxType!!!");
            }
            mIndexToNode.push_back(i1);
            mNbrMaps.push_back(std::map<IdxType, WeightType>{});
            mNodeToIndex.insert({i1, n1});
        }
        else
        {
            n1 = it1->second;
        }

        mNbrMaps[n0].insert({n1, w});
        mNbrMaps[n1].insert({n0, w});
    }

    void GetNbrs(std::vector<std::pair<NodeType, WeightType>> &nbrs,
                 const NodeType &i0)
    {
        auto it0 = mNodeToIndex.find(i0);
        if(it0==mNodeToIndex.end())
        {
            nbrs.clear();
        }
        else
        {
            IdxType n0 = it0->second;
            const std::map<IdxType, WeightType> &inbrs = mNbrMaps[n0];

            IdxType num = inbrs.size();
            nbrs.resize(num);
            IdxType count = 0;
            for(const std::pair<IdxType, WeightType> &nit: inbrs)
            {
                nbrs[count] = std::pair<NodeType, WeightType>(mIndexToNode[nit.first], nit.second);
                ++count;
            }
        }   
    }

    bool EdgeExists(const NodeType &i0, const NodeType &i1)
    {
        const auto& it0 = mNodeToIndex.find(i0);
        if(it0==mNodeToIndex.end()){
            return false;
        }
        const auto& it1 = mNodeToIndex.find(i1);
        if(it1==mNodeToIndex.end()){
            return false;
        }
        const std::map<IdxType, WeightType>& nbr0 = mNbrMaps[it0->second];
        if(nbr0.find(it1->second)==nbr0.end()){
            return false;
        }
        return true;
    }

    bool ComputeDijkShortestPath(std::unordered_map<NodeType, NodeType>& prevMap,
                                 std::unordered_map<NodeType, WeightType>& distMap,
                                 const NodeType &src)
    {
        const auto& it0 = mNodeToIndex.find(src);
        if(it0==mNodeToIndex.end()){
            return false;
        }
        std::vector<IdxType> prevMap_;
        std::vector<WeightType> distMap_;
        ComputeDijkShortestPathImpl(prevMap_, distMap_, it0->second);
        prevMap.clear();
        distMap.clear();

        const IdxType nv = mIndexToNode.size();
        for(IdxType iv=0; iv<nv; ++iv)
        {
            if(prevMap_[iv]==std::numeric_limits<IdxType>::max())
            {
                continue;
            }
            prevMap.insert({mIndexToNode[iv], mIndexToNode[prevMap_[iv]]});
            distMap.insert({mIndexToNode[iv], distMap_[iv]});
        }
        return true;
    }

    bool GetPath(std::vector<NodeType> &path,
                 const std::unordered_map<NodeType, NodeType>& prevMap,
                 const NodeType src,
                 const NodeType trg)
    {
        std::vector<NodeType> tmpPath;
        bool suc = true;
        size_t maxL = prevMap.size();
        size_t i = 0;
        NodeType cur = trg;
        while (i++<maxL)
        {
            tmpPath.push_back(cur);
            if(cur==src)
            {
                break;
            }
            auto prevIt = prevMap.find(cur);
            if(prevIt==prevMap.end()){
                suc = false;
                break;
            }
            cur = prevIt->second;
        }
        if(suc){
            std::reverse(tmpPath.begin(), tmpPath.end());
            std::swap(path, tmpPath);
        }
        return suc;
    }

    void ComputeSteinerTree(
        std::vector<std::pair<NodeType, NodeType>>& treeEdges,
        const std::vector<NodeType> &keyNodes)
    {
        IdxType num = keyNodes.size();
        std::vector<IdxType> keys(num);
        for(IdxType i=0; i<num; ++i)
        {
            auto it = mNodeToIndex.find(keyNodes[i]);
            LogAssert(it!=mNodeToIndex.end(), "failed find node");
            keys[i] = it->second;
        }
        std::vector<Edge> treeEdges_;
        ComputeSteinerTreeImpl(treeEdges_, keys);
        treeEdges.clear();
        for(const Edge &e: treeEdges_)
        {
            treeEdges.push_back({mIndexToNode[e.v1], mIndexToNode[e.v2]});
        }
    }

protected:

    template<typename KeyType, typename ValueType>
    struct GreaterCompare
    {
        bool operator()(const std::pair<KeyType, ValueType> &a, 
                        const std::pair<KeyType, ValueType> &b) const{
            return a.second > b.second;
        }
    };

    struct Edge
    {
        IdxType v1;
        IdxType v2;
        Edge(IdxType v1_, IdxType v2_)
        :v1(v1_), v2(v2_)
        {
        }

        bool operator==(Edge const &e) const {
            return e.v1==v1 && e.v2==v2;
        }
        bool operator!=(Edge const &e) const {
            return !operator==(e);
        }
    };

    struct EdgeHash
    {
        size_t operator()(Edge const &e) const
        {
            size_t seed = 0;
            seed ^= std::hash<IdxType>()(e.v1) + 0x9e3779b9 + (seed << 6) + (seed >> 2);
            seed ^= std::hash<IdxType>()(e.v2) + 0x9e3779b9 + (seed << 6) + (seed >> 2);
            return seed;
        }
    };

    bool ComputeDijkShortestPathImpl(std::vector<IdxType>& prevMap,
                                    std::vector<WeightType>& distMap,
                                    const IdxType &src)
    {
        std::priority_queue<std::pair<IdxType, WeightType>, 
            std::vector<std::pair<IdxType, WeightType>>, 
            GreaterCompare<IdxType, WeightType>> prique;

        const IdxType nv = mIndexToNode.size();
        prevMap.clear();
        prevMap.resize(nv, std::numeric_limits<IdxType>::max());
        distMap.clear();
        distMap.resize(nv, std::numeric_limits<WeightType>::max());

        prevMap[src] = src;
        distMap[src] = WeightType{0};

        std::pair<IdxType, WeightType> seed(src, WeightType{0});
        prique.push(seed);

        while (!prique.empty())
        {
            std::pair<IdxType, WeightType> cur = prique.top();
            prique.pop();

            if(distMap[cur.first] < cur.second){
                continue;
            }

            const std::map<IdxType, WeightType> &nbr = mNbrMaps[cur.first];
            for(const std::pair<IdxType, WeightType>& next: nbr)
            {
                WeightType dist = cur.second + next.second;
                if(dist < distMap[next.first])
                {
                    distMap[next.first] = dist;
                    prevMap[next.first] = cur.first;
                    prique.push({next.first, dist});
                }
            }
        }

        return true;
    }

    bool ComputeSteinerTreeImpl(std::vector<Edge> &treeEdges,
                                const std::vector<IdxType>& keyNodes)
    {
        const IdxType nk = keyNodes.size();
        const IdxType nv = mIndexToNode.size();

        std::vector<IdxType> fastprev(nv);
        std::iota(fastprev.begin(), fastprev.end(), IdxType{0});
        auto prevFun = [&fastprev](IdxType x)->IdxType
        {
            IdxType y = x;
            IdxType z = fastprev[y];
            if(y!=z)
            {
                std::stack<IdxType> deepstack;
                while(y!=z)
                {
                    deepstack.push(y);
                    y = z;
                    z = fastprev[z];
                }
                while(!deepstack.empty())
                {
                    y = deepstack.top();
                    deepstack.pop();
                    fastprev[y] = fastprev[fastprev[y]];
                }
            }
            return fastprev[x];
        };

        std::vector<IdxType> traceprev(nv, std::numeric_limits<IdxType>::max());

        std::vector<IdxType> rank(nv, IdxType{0});
        std::vector<uint8_t> marked(nv, 0);

        std::priority_queue<std::pair<Edge, WeightType>, 
            std::vector<std::pair<Edge, WeightType>>, 
            GreaterCompare<Edge, WeightType>> prique;
        // add key node neighbor to que
        for(IdxType ik=0; ik<nk; ++ik)
        {
            IdxType idx = keyNodes[ik];
            const std::map<IdxType, WeightType> &nbrs = mNbrMaps[idx];
            for(const std::pair<IdxType, WeightType> &inbr: nbrs)
            {
                prique.push({Edge(inbr.first, idx), inbr.second});
            }
            marked[prevFun(idx)] = 1;
        }

        std::unordered_set<Edge, EdgeHash> cuttedEdges;

        while(!prique.empty())
        {
            std::pair<Edge, WeightType> cur = prique.top();
            prique.pop();

            IdxType v1 = cur.first.v1;
            IdxType v2 = cur.first.v2;
            IdxType pv1 = prevFun(v1);
            IdxType pv2 = prevFun(v2);
            if(pv1!=pv2)
            {
                if(marked[pv1] && marked[pv2])
                {
                    IdxType p0 = v1;
                    IdxType p1 = traceprev[p0];
                    while(p1!=std::numeric_limits<IdxType>::max()){
                        cuttedEdges.insert(Edge(p1, p0));
                        p0 = p1;
                        p1 = traceprev[p1];
                    }

                    cuttedEdges.insert(cur.first);
                    p0 = v2;
                    p1 = traceprev[p0];
                    while(p1!=std::numeric_limits<IdxType>::max()){
                        cuttedEdges.insert(Edge(p1, p0));
                        p0 = p1;
                        p1 = traceprev[p1];
                    }
                }

                traceprev[v1] = v2;
                {
                    IdxType x = pv1;
                    IdxType y = pv2;
                    if(rank[x] > rank[y]) {
                        fastprev[y] = x;
                    }else{
                        fastprev[x] = y;
                    }
                    if(rank[x]==rank[y]){
                        rank[y]++;
                    }
                    if(marked[x] || marked[y])
                    {
                        marked[x] = 1;
                        marked[y] = 1;
                    }
                }

                const std::map<IdxType, WeightType> &nbrs = mNbrMaps[v1];
                for(const std::pair<IdxType, WeightType> &inbr: nbrs)
                {
                    prique.push({Edge(inbr.first, v1), inbr.second + cur.second});
                }
            }
        }

        treeEdges.clear();
        for(const Edge& e: cuttedEdges)
        {
            treeEdges.push_back(e);
        }
        return true;
    }

};


```

