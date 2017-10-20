---
layout: post
title:  "CATansaction"
date:   2017-10-14 0:03:01 +0800
categories: iOS
tags: [iOS, Core Animation]
---

* TOC
{:toc}

# Core Animation 

------

> Core Animation provides high frame rates and smooth animations without burdening the CPU and slowing down your app. ------ [Core Animation Doc][apple-doc-core-animation] 

<br>

`核心动画`提供高帧数和流畅的动画， 且不会增加CPU负担，使应用卡顿。

<br>

> Most of the work required to draw each frame of an animation is done for you. You configure a few animation parameters (such as the start and end points) and tell Core Animation to start. Core Animation does the rest, handing most of the work off to dedicated graphics hardware to accelerate rendering. ------ [Core Animation Doc][apple-doc-core-animation] 

<br>

大部分绘制动画的每一帧的工作已经为你完成，你只需要配置一些动画参数，如起始点，终止点，给予`核心动画`开始指令。`核心动画`会完成剩余的所有事情， 将大部分工作交由专门的图形硬件加速渲染。

<br>

# Core Animation Basic

------

> Core Animation provides a general purpose system for animating views and other visual elements of your app. Core Animation is not a replacement for your app’s views. Instead, it is a technology that integrates with views to provide better performance and support for animating their content. It achieves this behavior by caching the contents of views into bitmaps that can be manipulated directly by the graphics hardware. 

<br>

`Core Animation` 是一个通用的为App中的**视图** 和 其他**可视元素**添加动画的系统。`Core Animation` 不是视图的替代品。而是一种与视图集成进而提供更好的性能以及支持动画视图内容。`Core Animation` 通过缓存视图内容为可以直接被图形硬件操作的 **位图** 来完成这个行为。

<br>

> In some cases, this caching behavior might require you to rethink how you present and manage your app’s content, but most of the time you use Core Animation without ever knowing it is there. In addition to caching view content, Core Animation also defines a way to specify arbitrary visual content, integrate that content with your views, and animate it along with everything else.

<br>

有的时候，这种缓存行为可能需要让你去斟酌如何去展示和管理你的应用的内容，但是大部分时候使用`Core Animation` 不需要在意太多细节。为了缓存视图内容，`Core Animation` 也制定了一种描述任意可视内容的方式，将这种可视内容集成入你的视图，并与其他视图元素一起动画。

<br>

------

> You use Core Animation to animate changes to your app’s views and visual objects. Most changes relate to modifying the properties of your visual objects. For example, you might use Core Animation to animate changes to a view’s position, size, or opacity. When you make such a change, Core Animation animates between the current value of the property and the new value you specify. 

<br>

你使用 `Core Animation` 来产生动画变化你的应用中的视图和可视对象。大部分改变通过修改可视图像的属性。如，你可能用 `Core Animation` 来动画改变一个视图的位置，尺寸，或者透明度。当你做出一些改变， `Core Animation` 将会产生由当前值到目标值的动画。

<br>

> You would typically not use Core Animation to replace the content of a view 60 times a second, such as in a cartoon. Instead, you use Core Animation to move a view’s content around the screen, fade that content in or out, apply arbitrary graphics transformations to the view, or change the view’s other visual attributes.

<br>

你不应该使用 `Core Animation` 替换视图内容60次一秒（卡通的动画方式）。你应该使用`Core Animation`来在屏幕中移动视图内容，淡入淡出视图内容，对视图应用任意图形变化，或者视图其他的可是属性。

<br>

# Layers Provide the Basis for Drawing and Animations

> Layer objects are 2D surfaces organized in a 3D space and are at the heart of everything you do with Core Animation. Like views, layers manage information about the geometry, content, and visual attributes of their surfaces. Unlike views, layers do not define their own appearance. A layer merely manages the state information surrounding a bitmap. The bitmap itself can be the result of a view drawing itself or a fixed image that you specify. For this reason, the main layers you use in your app are considered to be model objects because they primarily manage data. This notion is important to remember because it affects the behavior of animations.

<br>

Layer 对象是三维空间内的二维平面，并且是`Core Animation`的核心。象视图一样，layers 管理者几何信息，内容，和平面的视觉属性。不象视图，layers不会定义拥有者的显示效果一个Layer只是管理位图的状态信息。这个位图可以是一个视图绘制自身的结果或者是一张固定的图片。因此，你使用的主要的layers可以看做是`模型对象（model objects）`，因为他们主要是管理数据。这点很重要，因为他影响着动画的行为。

<br>

## The Layer-Based Drawing Model

> Most layers do not do any actual drawing in your app. Instead, a layer captures the content your app provides and caches it in a bitmap, which is sometimes referred to as the backing store. When you subsequently change a property of the layer, all you are doing is changing the state information associated with the layer object. When a change triggers an animation, Core Animation passes the layer’s bitmap and state information to the graphics hardware, which does the work of rendering the bitmap using the new information, as shown in Figure 1-1. Manipulating the bitmap in hardware yields much faster animations than could be done in software.

<br>

大部分layers不会做任何绘制，而是抓取你的app提供的内容，缓存为位图，有时会被称为后备存储器。当你依次改变一个layer的属性，你所做的只是改变了关联在这个layer上的状态信息。当一个改变触发了动画，`Core Animation` 传递这个layer的位图还有状态信息到图形硬件，图形硬件负责根据新的信息渲染位图。如图1-1所示，硬件操作位图产出比实用软件更快的动画。

<br>

![img](http://img)

> Because it manipulates a static bitmap, layer-based drawing differs significantly from more traditional view-based drawing techniques. With view-based drawing, changes to the view itself often result in a call to the view’s drawRect: method to redraw content using the new parameters. But drawing in this way is expensive because it is done using the CPU on the main thread. Core Animation avoids this expense by whenever possible by manipulating the cached bitmap in hardware to achieve the same or similar effects.

<br>

 因为它（Layer）操作静态的位图，`基于layer的绘制`迥异于传统的`基于视图的绘制技术`。基于视图的绘制，改变视图自身经常造成调用视图的`drawRect:`方法来使用新的参数重绘内容，但是这样会使代价昂贵（使用CPU在主线程进行绘制），`Core Animation`通过硬件操作缓存的位图来完成相同或者相似的作用，尽可能的避免了这种损耗。

<br>

> Although Core Animation uses cached content as much as possible, your app must still provide the initial content and update it from time to time. There are several ways for your app to provide a layer object with content, which are described in detail in [Providing a Layer’s Contents](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/CoreAnimation_guide/SettingUpLayerObjects/SettingUpLayerObjects.html#//apple_ref/doc/uid/TP40004514-CH13-SW4).

<br>

 虽然`Core Animation`尽可能的使用缓存的内容，你的app任然必须提供初始化内容并且实时更新。你的app有多种方式提供内容给layer对象, 详见[提供layer的内容](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/CoreAnimation_guide/SettingUpLayerObjects/SettingUpLayerObjects.html#//apple_ref/doc/uid/TP40004514-CH13-SW4)

<br>

## Layer-Based Animations

> The data and state information of a layer object is decoupled from the visual presentation of that layer’s content onscreen. This decoupling gives Core Animation a way to `interpose` itself and animate the change from the old state values to new state values. For example, changing a layer’s position property causes Core Animation to move the layer from its current position to the newly specified position. Similar changes to other properties cause appropriate animations. Figure 1-2 illustrates a few of the types of animations you can perform on layers. For a list of layer properties that trigger animations, see [Animatable Properties](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/CoreAnimation_guide/AnimatableProperties/AnimatableProperties.html#//apple_ref/doc/uid/TP40004514-CH11-SW1).

<br>

layer对象的数据和状态信息与屏幕上layer的内容的视觉显示解耦。这种解耦使 `Core Animation` 可以调节自身以及从老的状态改变到新的状态。如,改变一个layer的位置属性会造成 `Core Animation`移动layer从当前的位置到所指定的新位置。相同的，改变其他属性会造成相应的适当的动画。图1-2列举了一些列类型你可以在layer上执行的动画。会触发动画的layer属性列表，详见[可动画属性](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/CoreAnimation_guide/AnimatableProperties/AnimatableProperties.html#//apple_ref/doc/uid/TP40004514-CH11-SW1)

<br>

![img](http://img)

> During the course of an animation, Core Animation does all of the frame-by-frame drawing for you in hardware. All you have to do is specify the start and end points of the animation and let Core Animation do the rest. You can also specify custom timing information and animation parameters as needed; however, Core Animation provides suitable default values if you do not.

<br>

在动画过程中，`Core Animation`使用硬件你做每一帧的绘制。所有你需要做的是指定开始和结束点，让`Core Animation`做剩余的内容。你也可以按需制定动画时间，和动画参数（`Core Animation`会提供适合的默认值）。

<br>

>For more information about how to initiate animations and configure animation parameters, see [Animating Layer Content](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/CoreAnimation_guide/CreatingBasicAnimations/CreatingBasicAnimations.html#//apple_ref/doc/uid/TP40004514-CH3-SW1).

<br>

关于更多的初始化动画和配置动画参数，详见[Animating Layer Content](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/CoreAnimation_guide/CreatingBasicAnimations/CreatingBasicAnimations.html#//apple_ref/doc/uid/TP40004514-CH3-SW1)

<br>


# Layer Trees Reflect Different Aspects of the Animation State

> `Layer Tree` 映射着动画状态的不同部分。

-------

### model layer tree

> Objects in the model layer tree (or simply “layer tree”) are the ones your app interacts with the most. The objects in this tree are the model objects that store the target values for any animations. Whenever you change the property of a layer, you use one of these objects.

> `model layer tree` 是与你的应用交互最多的部分。 `model layer tree` 中的对象是为各种动画存储着目标值的`模型对象（model objects）`，你修改layer的任一属性，都是在使用`模型对象`。

### presentation tree

> Objects in the presentation tree contain the in-flight values for any running animations. Whereas the layer tree objects contain the target values for an animation, the objects in the presentation tree reflect the current values as they appear onscreen. You should never modify the objects in this tree. Instead, you use these objects to read current animation values, perhaps to create a new animation starting at those values. 

> `presentation tree` 中的对象， 包含动画中在变化的值。`layer tree` 包含动画的目标值，`presentation tree` 中的对象对应屏幕上显示的当前值。不应该去修改`presentation tree`中的值，而是读取当前动画的参数，或许根据当前值开始一个新的动画。

### render tree

> Objects in the render tree perform the actual animations and are private to Core Animation.

> `render tree` 中的对象执行真实的动画，在`Core Animation`中是私有的。

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

# iOS系统中的CATransaction的应用

## 转场
TODO: 转场中使用的调用栈

### 嵌套cell 中有tableView

## 耗费性能的动画

### 闪动
### 阴影

# 引用
------

[CATansaction Doc][apple-doc-catransaction]

[Core Animation Program Guide][apple-doc-core-animation-programming-guide]

[apple-doc-catransaction]: https://developer.apple.com/documentation/quartzcore/catransaction
[apple-doc-core-animation]: https://developer.apple.com/documentation/quartzcore
[apple-doc-caanimationgroup]: https://developer.apple.com/documentation/quartzcore/caanimationgroup

[apple-doc-core-animation-programming-guide]: https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/CoreAnimation_guide/CoreAnimationBasics/CoreAnimationBasics.html