---
layout: post
title:  "多层有向图布局算法 (Gansner)"
date:   2019-02-03 00:35:23 +0800
categories: reading
tags: [reading, 算法]
---

# 基本步骤
```shell
-dot1_rank
  - collapse_cluster # locally rank cluster
    - dot1_rank
  - class1 # 合并cluster's node (基于Union Find) 但是group根据cluster来分类 增加 virual node 和 aux edge 为 intercluster edge
  - decompose # 将图重新转化为 多个连通分量(connected component)
  - acyclic # 去环
  - rank1 # 将转化后的图 rank
  - expand_ranksets # 展开 之前collaspe 的 group
```
# 分析
`rank1` 以及 `expand_ranksets` 之后, 才能决定每一层有多少个节点,** 需要优化单层节点个数, 以及根据cluster的情况**, 调整纵横比
