---
layout: post
title:  "CATansaction"
date:   2017-10-14 0:03:01 +0800
categories: iOS
tags: [iOS, Core Animation, 资料]
---

# Core Animation 

------

> Core Animation provides high frame rates and smooth animations without burdening the CPU and slowing down your app. (`核心动画`提供高帧数和流畅的动画， 且不会增加CPU负担，使应用卡顿。) ------ [Core Animation Doc][apple-doc-core-animation] 

> Most of the work required to draw each frame of an animation is done for you. You configure a few animation parameters (such as the start and end points) and tell Core Animation to start. Core Animation does the rest, handing most of the work off to dedicated graphics hardware to accelerate rendering. （大部分绘制动画的每一帧的工作已经为你完成，你只需要配置一些动画参数，如起始点，终止点，给予`核心动画`开始指令。`核心动画`会完成剩余的所有事情， 将大部分工作交由专门的图形硬件加速渲染）------ [Core Animation Doc][apple-doc-core-animation] 

# CAAnimationGroup

------

> An object that provides an animated transition between a layer's states (提供layer的两个`状态`之间变化的动画转场的对象) ------ [CAAnimationGroup Doc][apple-doc-caanimationgroup]

# CATansaction

------

> A mechanism for grouping multiple `layer-tree` operations into `atomic` updates to the `render tree`. (一种合并多个`layer树`操作为一个`原子操作`来更新`渲染树`的机制) ------- [CATansaction Doc][apple-doc-catransaction]

`事务（transaction）`通常的作用是统一一系列强关联性的操作，让这些操作**要么一起成功，要么一起失败**，也就是`原子性（atomic）`。

CATransaction支持`嵌套(nested)`，支持两种事务，`隐式事务(implicit transactions)`和`显示事务(explicit transactions)`

## 隐式事务(implicit transactions)

> Implicit transactions are created automatically when the layer tree is modified by a thread without an active transaction and are committed automatically when the thread's runloop next iterates.

## 显示事务(explicit transactions)

>  Explicit transactions occur when the the application sends the CATransaction class a `begin()` message before modifying the layer tree, and a `commit()` message afterwards.

> During a transaction you can temporarily acquire a recursive spin lock for managing property atomicity.(进行事务时，你可以临时获取一个递归的自旋锁来管理属性的原子性。) 

TODO: 可以多线程？
详见 lock unlock API
有什么作用？

### 例程
```swift
let transitioningLayer = CALayer()
     
// Outer transaction animates `opacity` to 0 over 2 seconds
CATransaction.begin()
CATransaction.setAnimationDuration(2)
CATransaction.setCompletionBlock {
    transitioningLayer.removeFromSuperlayer()
}
    
transitioningLayer.opacity = 0
     
// Inner transaction animates scale to (3, 3, 3) over 1 second
CATransaction.begin()
CATransaction.setAnimationDuration(1)
     
transitioningLayer.transform = CATransform3DMakeScale(3, 3, 3)
     
CATransaction.commit() // Commits inner transaction
CATransaction.commit() // Commits outer transaction
```
# 名称解释

## layer tree

## render tree

# 引用
------

[CATansaction Doc][apple-doc-catransaction]


[apple-doc-catransaction]: https://developer.apple.com/documentation/quartzcore/catransaction
[apple-doc-core-animation]: https://developer.apple.com/documentation/quartzcore
[apple-doc-caanimationgroup]: https://developer.apple.com/documentation/quartzcore/caanimationgroup