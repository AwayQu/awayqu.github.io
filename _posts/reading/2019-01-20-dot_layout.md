---
layout: post
title:  "单层有向图布局算法 (Gansner)"
date:   2019-01-20 00:35:23 +0800
categories: reading
tags: [reading, 算法]
---


# 单层有向图布局算法 (Gansner)

> 这种布局方式会把clust忽略, 后续的苏散发版本是有升级对于clust的优化,但是好像效果还是不是很理想


```
procedure draw_graph() 
begin
rank();
ordering(); 
position(); 
make_splines();
end
```

## rank

> rank, len, minlen, subset(rank [same sink source])
> node : v , edge : e

* λ(v) rank of edge

* l ( e )  length l(e) of e = ( v , w ) 
* l ( e ) = λ ( w ) − λ ( v )

*  δ ( e )  represents some given minimum length constraint default:1
*  l ( e ) ≥ δ ( e ) ≥ 1

* ω(e) weight of edge

*  Smax ,Smin ,S0 ,S1 , . . . , Sk !subset V. A sets of nodes that must be placed together on the maximum, minimum, or same rank, respectively.

### 第一步 处理图生成无环有向图

#### 初始化 rank 以及 weight
* 非空子图合并为一个 virtual node. 
> each of the nonempty sets Smax ,Smin ,S0 , . . . ,Sk is temporarily merged into one node. 
* 合并多条edge为一条 获取 weight. 
> multiple edges are merged into one edge whose weight is the sum of the weights of the merged edges 

#### 打破图中的环 (acyclic)

* 发现环,并且翻转edge. 
> preprocessing step detects cycles and breaks them by reversing certain edges [RDM]

* 从sink 或者 source node 开始查找打破环, 并且开始将 edges 分为两类 tree edges 和 non-tree edges. 
> Edges are searched in the ‘‘natural order’’ of the graph input, starting from some source or sink nodes if any exist. Depth-first search partitions edges into two sets: tree edges and non-tree edges [AHU]. 

* tree edges 定义了部分nodes的顺序(rank)
> The tree defines a partial order on nodes.

*  将 non-tree edges 分为三类 cross edges, forward edges, and back edges.
> Cross edges connect unrelated nodes in the partial order. (cross edges 连接了 非 tree edges 中的node)
  Forward edges connect a node to some of its descendants. (forward edges 连接到一些他自己的子孙node)
  Back edges connect a descendant to some of its ancestors.  (back edges 连接 子孙node到他的父node)

  * 加入 foward edge 或者 back edges 不会产生环, 因为可以翻转
  > It is clear that adding forward and cross edges to the partial order does not create cycles. Because reversing back edges makes them into forward edges, all cycles are broken by this procedure.

* 为了减少翻转, 将拆分出部分强连通图, 计算遍历 强连通图 并且计数的时候, 计数最多的边, 最需要翻转.
> It seems reasonable to try to reverse a smaller or even minimal set of edges. One difficulty is that finding a minimal set (the ‘‘feedback arc set’’ problem) is NP-complete [EMW] [GJ]. More important, this would probably not improve the drawings. We implemented a heuristic to reverse edges that participate in many cycles. The heuristic takes one non-trivial strongly connected component at a time, in an arbitrary order. Within each component, it counts the number of times each edge forms a cycle in a depth-first traversal. An edge with a maximal count is reversed. This is repeated until there are no more non-trivial strongly connected components.

* 为了保证 Sk, Smax and Smin rank一致. 翻转所有的Smax的out-edges,以及所有的Smin的in-edges, 并且构造δ = 0的临时edge(有什么用?好像翻转就足以保证满足后续条件), 使满足 λ(Smin ) ≤ λ(v) ≤ λ(Smax )
> This property is ensured by reversing out-edges of Smax and in-edges of S min . Also, for all nodes v with no in-edge, we make a temporary edge ( S min , v ) with δ = 0, and for all nodes v with no out-edge, we make a temporary edge ( v , S max ) with δ = 0. Thus, λ(Smin ) ≤ λ(v) ≤ λ(Smax ) for all v.

`in-edge` w -> v  则 是node v的in-edge
`out-edge` w -> v  则是node w的out-edge

#### 保持edge路径短
> Principle A3 prescribes making short edges.

*可以转化为问题:*
min Σ ω ( v , w ) ( λ ( w ) − λ ( v ) )  
subject to: λ(w) − λ(v) ≥ δ(v,w) ∀ (v,w) memberE

### 简化连接网络

给出如下定义
`feasible rank` 所有的edge 满足 l(e) ≥ δ(e) . 
`slack` edge的属性 代表 slack(e) = l(e) - δ(e)

也就是说`Σslack(e)`越小 这个`feasible rank`越tight(紧致)

`spanning tree` 带有rank的tree

`a feasible spanning tree` 带有 `feasible rank`的 `spanning tree`

`integer cut value` 将一个 `feasible spanning tree` 删除 `tree edge` 拆分为两个 连接子图`connected component` integer cut value = sum(ω(taile)) + sum(ω(tree edge))

`anti-cycling` 反循环

*network simple algorithm `rank`*

```c
procedure rank()
    feasible_tree(); // 构造  an initial feasible spanning tree
    while (e = leave_edge()) ≠ nil do // 获取negetive integer cut value
        f = enter_edge(e); //
        exchange(e,f);
    end
    normalize(); // 整流规则化
    balance();  // 平衡树
    end
```

*spanning tree algorithm `feasible tree`*
```c
procedure feasible_tree() 
    init_rank(); // 初始化rank
    while tight_tree() < |V| do // 找到最大的 tight tree 并且返回node数
        e = a non-tree edge incident on the tree
            with a minimal amount of slack;
        delta = slack(e);
        if incident node is e.head then delta = -delta; 
        for v in Tree do v.rank = v.rank + delta;
    end
    init_cutvalues();
end
```