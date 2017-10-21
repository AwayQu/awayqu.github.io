---
layout: post
title:  "Core Animation Programming Guide（概括）"
date:   2017-10-21 19:44:01 +0800
categories: iOS
tags: [iOS, Core Animation]
---

# Layer & Core Animation 技术的目的

--------

`Layer` & `Core Animation`技术的目的，**为了丝滑的用户体验（响应快速，高帧数以及动画流畅）**。

因而**主线程不能有太多耗时任务**。

主线程（`Main Thread`）有很多任务需要执行，`事件处理`, `视图绘制更新`，`回调派发`等等，**其中`视图绘制更新`显然是其中比较耗时的任务，所以对其进行优化**。

优化性能常见的考虑方向有`优化计算(减少多余计算)`,`缓存数据`等方式。为了做到这些优化，**需要建立数据模型，在iOS中是`Layer`**。


所以`Layer`存储着位图的状态信息，这个状态信息包含`仿射变化矩阵（affine transform matrix）`,`位置（position）`，`坐标系统以及尺寸（bounds）`，`颜色（color）`，`缩放(scale)`等等，而无关真实的显示。

**有了这一层数据模型`Layer`之后，才能基于它做一些性能优化**。

当视图（view）与显示相关的数据信息发生变化，如：`位置（position）`，`颜色(color)`, `内容(content)`，如果仅是`位图的状态信息`发生变更, 那么就可以不需要重新绘制视图，而使用`Core Animation`（使用GPU计算渲染变换后的位图）。只有位图自身需要发生变化时，才需要进行重新绘制（使用CPU重新绘制）。

由此，**1.减少重新绘制的调用 2.通过缓存的位图调用GPU处理生成新位图**。

**将计算从主线程解放出来，同时减少重绘调用**。 

# 引入的问题

--------

`数据模型(Layer)`的功能执行需要对位图进行缓存，**增加了内存占用**。

动画的执行，以及视图的更新，会需要**异步等待GPU对于位图的处理完成**，`复杂度`提高，容易引入问题。

<!-- 首先视图绘制必须要在主线程中完成（ps:也许是0.0） -->
<!-- 

需要后续查的了解的点：
1.drawRect 使用CPU？
2.绘制视图到底是GPU负责还是CPU负责
3.GPU的工作
4.绘制能不能在子线程执行

 -->

# 引用

--------

[Core Animation Program Guide][apple-doc-core-animation-programming-guide]

[apple-doc-core-animation-programming-guide]: https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/CoreAnimation_guide/CoreAnimationBasics/CoreAnimationBasics.html





