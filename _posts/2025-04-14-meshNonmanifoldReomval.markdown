---
layout: post
title:  三角网非流形移除
date:   2025-02-27 15:30:00 +0300
tags:   ComputerGraphics
description:  mesh处理
---

# [简介](#简介)

在处理三角网的过程中，经常会用到三角网的半边结构，然而半边数据结构的前提假设是三角网符合二流形。那么不符合二流形假设会使得处理过程出现错误、崩溃等等。

一般使用Poisson重建算法得到的mesh是保证二流形的，但是其面片数量过多、过密会使得展示压力变大，所以通常会使用简化算法使得面片数量合理下降。简化算法会一定程度上产生非流形，会使得后续关于mesh的处理变得不稳定。

# [图](#图)
![]({{ site.baseurl }}/images/meshNonmanifoldRemoval-001.png)    

1. 顶点图：  
将mesh的顶点视为一个图(graph)的顶点，三角面片的边视为图(graph)的边(edge)，这就是mesh的顶点图。

2. 对偶图：  
将mesh的面片视为一个图(graph)的顶点，面片与另一个面片共享的边视为图(graph)的边(edge)，这就是顶点图的对偶图。

# [非流形顶点](非流形顶点)

![]({{ site.baseurl }}/images/meshNonmanifoldRemoval-002.png)    

一个顶点的邻域面片构成的对偶图是由多个互不相连的连通子图。

# [非流形面片](非流形面片)

![]({{ site.baseurl }}/images/meshNonmanifoldRemoval-003.png)  
![]({{ site.baseurl }}/images/meshNonmanifoldRemoval-004.png)  
一条边被超两面片共享或共享该边的两个面片在该边顺序一致。

# [去除非流形顶点](去除非流形顶点)

1. 构造mesh的顶点图与对偶图；    
2. 将顶点邻域面片分组，按对偶图连通性分组；    
3. 分组数量大于等于2，则为非流形顶点；    
4. 增加新顶点，位置信息可以一致，每组面片的顶点id重新映射到新顶点id；   

```cpp
// only add verts, not add faces
template<typename Real, typename IdxType>
void RemoveNonmanifoldVert(Mesh<Real, IdxType> &mesh)
{
    const IdxType nf = mesh.mFaces.size();
    const IdxType nv = mesh.mVerts.size();

    std::unordered_map<EdgeKey<false, IdxType>, 
        std::vector<IdxType>, EdgeKey<false, IdxType>, EdgeKey<false, IdxType>> edgeNbrFids;
    ConstructTriangleDualGraph(edgeNbrFids, mesh.mFaces);

    std::unordered_map<IdxType, std::vector<IdxType>> vertNbrVids;
    ConstructVertCenterGraph(vertNbrVids, mesh.mFaces);

    std::vector<std::vector<std::vector<IdxType>>> nonmanifoldVertLocalInfos;
    std::map<IdxType, IdxType> nonmanifoldVertInfoMap;

    for(const auto &vit: vertNbrVids)
    {
        const IdxType cvid = vit.first;
        const std::vector<IdxType> &vids = vit.second;
        std::vector<std::set<IdxType>> fsetgroups;
        std::map<IdxType, IdxType> fid2fgid; 
        IdxType count = 0;
        for(IdxType vid: vids)
        {
            EdgeKey<false, IdxType> e(cvid, vid);
            const auto eit = edgeNbrFids.find(e);
            LogAssert(eit!=edgeNbrFids.end(), "failed find edge");
            IdxType f0 = eit->second.front();
            auto f0it = fid2fgid.find(f0);
            IdxType fgid = 0;
            if(f0it==fid2fgid.end())
            {
                fgid = fsetgroups.size();
                fsetgroups.push_back(std::set<IdxType>{});
                std::set<IdxType>& fset = fsetgroups.back();
                fset.insert(f0);
                fid2fgid.insert({f0, fgid});
                ++count;
            }
            else
            {
                fgid = f0it->second;
            }

            std::set<IdxType> &f0set = fsetgroups[fgid];
            for(IdxType fid: eit->second)
            {
                if(fid==f0){continue;}
                auto fidit = fid2fgid.find(fid);
                if(fidit==fid2fgid.end())
                {
                    f0set.insert(fid);
                    fid2fgid.insert({fid, fgid});
                }
                else
                {
                    if(fidit->second != fgid)
                    {
                        std::set<IdxType> &fidset = fsetgroups[fidit->second];
                        for(IdxType fjj: fidset){
                            f0set.insert(fjj);
                            fid2fgid[fjj] = fgid;
                        }
                        fidset.clear();
                        --count;
                    }
                }
            }
        }

        IdxType faceGroupCount = 0;
        for(std::set<IdxType> &fg: fsetgroups)
        {
            if(fg.empty()){continue;}
            faceGroupCount++;
        }
        LogAssert(count==faceGroupCount, "nonmanifold count error");

        if(faceGroupCount >=2 )
        {
            nonmanifoldVertInfoMap.insert({cvid, nonmanifoldVertLocalInfos.size()});
            nonmanifoldVertLocalInfos.push_back(std::vector<std::vector<IdxType>>{});
            std::vector<std::vector<IdxType>> &localInfo = nonmanifoldVertLocalInfos.back();
            for(std::set<IdxType> &fg: fsetgroups)
            {
                if(fg.empty()){continue;}
                localInfo.push_back(std::vector<IdxType>{});
                for(IdxType fid: fg){
                    localInfo.back().push_back(fid);
                }
            }
        }
    }

    for(const auto &nonIt: nonmanifoldVertInfoMap)
    {
        IdxType vid = nonIt.first;
        vec3<Real> pos = mesh.mVerts[vid];
        const std::vector<std::vector<IdxType>>& fgroups = nonmanifoldVertLocalInfos[nonIt.second];
        IdxType nfg = fgroups.size();
        for(IdxType ifg=1; ifg<nfg; ++ifg)
        {
            IdxType trgvid = mesh.mVerts.size();
            mesh.mVerts.push_back(pos);
            for(IdxType fid: fgroups[ifg])
            {
                vec3<IdxType> & f = mesh.mFaces[fid];
                for(IdxType ii=0; ii<3; ++ii)
                {
                    if(f[ii]==vid)
                    {
                        f[ii] = trgvid;
                        break;
                    }
                }
            }
        }
    }

    spdlog::info("nonmanifold vert count: {}", nonmanifoldVertLocalInfos.size());
}
```

# [去除非流形边](去除非流形边)

1. 构造mesh的顶点图与对偶图； 
2. 根据对偶图将非流形边提取出来；    
3. 将每个非流形边视为两个非流形顶点；      
4. 将非流形顶点邻域面片分组，按对偶图连通性分组；   
5. 增加新顶点，位置信息可以一致，每组面片的顶点id重新映射到新顶点id；   

```cpp
// only add verts, not add faces
template<typename Real, typename IdxType>
void RemoveNonmanifoldEdge(Mesh<Real, IdxType> &mesh)
{
    const IdxType nf = mesh.mFaces.size();
    const IdxType nv = mesh.mVerts.size();

    std::unordered_map<EdgeKey<false, IdxType>, 
        std::vector<IdxType>, EdgeKey<false, IdxType>, EdgeKey<false, IdxType>> edgeNbrFids;
    ConstructTriangleDualGraph(edgeNbrFids, mesh.mFaces);

    std::unordered_map<IdxType, std::vector<IdxType>> vertNbrVids;
    ConstructVertCenterGraph(vertNbrVids, mesh.mFaces);

    std::unordered_set<EdgeKey<false, IdxType>,
            EdgeKey<false, IdxType>, EdgeKey<false, IdxType>> needSplitEdge;

    for(const auto &eit: edgeNbrFids)
    {
        const EdgeKey<false, IdxType> &e = eit.first;
        std::unordered_set<EdgeKey<true, IdxType>,
            EdgeKey<true, IdxType>, EdgeKey<true, IdxType>> eset;

        const std::vector<IdxType> &fids = eit.second;
        for(IdxType fid: fids)
        {
            const vec3<IdxType> &f = mesh.mFaces[fid];
            for(IdxType ii=0; ii<3; ++ii)
            {
                if((e.V[0]==f[ii] && e.V[1]==f[(ii+1)%3]) ||
                    (e.V[1]==f[ii] && e.V[0]==f[(ii+1)%3]))
                {
                    eset.insert(EdgeKey<true>(f[ii], f[(ii+1)%3]));
                    break;
                }
            }
        }
        if(eset.size()<fids.size())
        {
            needSplitEdge.insert(e);
        }
    }

    std::set<IdxType> uniqueVset;
    for(const auto &eit: needSplitEdge)
    {
        uniqueVset.insert(eit.V[0]);
        uniqueVset.insert(eit.V[1]);
    }

    std::vector<std::vector<std::vector<IdxType>>> nonmanifoldVertLocalInfos;
    std::map<IdxType, IdxType> nonmanifoldVertInfoMap;

    for(IdxType uqvid: uniqueVset)
    {
        const auto &vit = vertNbrVids.find(uqvid);
        LogAssert(vit!=vertNbrVids.end(), "failed find vert");
        const IdxType cvid = vit->first;
        const std::vector<IdxType> &vids = vit->second;
        std::vector<std::set<IdxType>> fsetgroups;
        std::map<IdxType, IdxType> fid2fgid; 
        IdxType count = 0;
        for(IdxType vid: vids)
        {
            EdgeKey<false, IdxType> e(cvid, vid);
            if(needSplitEdge.find(e)!=needSplitEdge.end()){
                continue;
            }

            const auto eit = edgeNbrFids.find(e);
            LogAssert(eit!=edgeNbrFids.end(), "failed find edge");
            IdxType f0 = eit->second.front();
            auto f0it = fid2fgid.find(f0);
            IdxType fgid = 0;
            if(f0it==fid2fgid.end())
            {
                fgid = fsetgroups.size();
                fsetgroups.push_back(std::set<IdxType>{});
                std::set<IdxType>& fset = fsetgroups.back();
                fset.insert(f0);
                fid2fgid.insert({f0, fgid});
                ++count;
            }
            else
            {
                fgid = f0it->second;
            }

            std::set<IdxType> &f0set = fsetgroups[fgid];
            for(IdxType fid: eit->second)
            {
                if(fid==f0){continue;}
                auto fidit = fid2fgid.find(fid);
                if(fidit==fid2fgid.end())
                {
                    f0set.insert(fid);
                    fid2fgid.insert({fid, fgid});
                }
                else
                {
                    if(fidit->second != fgid)
                    {
                        std::set<IdxType> &fidset = fsetgroups[fidit->second];
                        for(IdxType fjj: fidset){
                            f0set.insert(fjj);
                            fid2fgid[fjj] = fgid;
                        }
                        fidset.clear();
                        --count;
                    }
                }
            }
        }

        IdxType faceGroupCount = 0;
        for(std::set<IdxType> &fg: fsetgroups)
        {
            if(fg.empty()){continue;}
            faceGroupCount++;
        }
        LogAssert(count==faceGroupCount, "nonmanifold count error");

        if(faceGroupCount >=2 )
        {
            nonmanifoldVertInfoMap.insert({cvid, nonmanifoldVertLocalInfos.size()});
            nonmanifoldVertLocalInfos.push_back(std::vector<std::vector<IdxType>>{});
            std::vector<std::vector<IdxType>> &localInfo = nonmanifoldVertLocalInfos.back();
            for(std::set<IdxType> &fg: fsetgroups)
            {
                if(fg.empty()){continue;}
                localInfo.push_back(std::vector<IdxType>{});
                for(IdxType fid: fg){
                    localInfo.back().push_back(fid);
                }
            }
        }
    }

    for(const auto &nonIt: nonmanifoldVertInfoMap)
    {
        IdxType vid = nonIt.first;
        vec3<Real> pos = mesh.mVerts[vid];
        const std::vector<std::vector<IdxType>>& fgroups = nonmanifoldVertLocalInfos[nonIt.second];
        IdxType nfg = fgroups.size();
        for(IdxType ifg=1; ifg<nfg; ++ifg)
        {
            IdxType trgvid = mesh.mVerts.size();
            mesh.mVerts.push_back(pos);
            for(IdxType fid: fgroups[ifg])
            {
                vec3<IdxType> & f = mesh.mFaces[fid];
                for(IdxType ii=0; ii<3; ++ii)
                {
                    if(f[ii]==vid)
                    {
                        f[ii] = trgvid;
                        break;
                    }
                }
            }
        }
    }

    spdlog::info("nonmanifold edge count: {}", needSplitEdge.size());
}
```



