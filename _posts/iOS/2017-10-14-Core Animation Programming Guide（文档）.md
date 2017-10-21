---
layout: post
title:  "Core Animation Programming Guide（文档）"
date:   2017-10-14 0:03:01 +0800
categories: iOS
tags: [iOS, Core Animation]
---

* TOC
{:toc}

# Core Animation 简介

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

大部分layers不会做任何绘制，而是抓取你的app提供的内容，缓存为位图，有时会被称为后备存储器。当你依次改变一个layer的属性，你所做的只是改变了关联在这个layer上的状态信息。当一个改变触发了动画，`Core Animation` 传递这个layer的位图还有状态信息到图形硬件，图形硬件负责根据新的信息渲染位图。如图1-1所示，硬件操作位图产出比使用软件更快的动画。

<br>

**图1-1**:

![image.png]({{ site.img_url }}imgs/iOS/basics_layer_rendering_2x.png)

<br>

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

**图1-2**

![img]({{ site.img_url }}imgs/iOS/basics_animation_types_2x.png)

<br>

> During the course of an animation, Core Animation does all of the frame-by-frame drawing for you in hardware. All you have to do is specify the start and end points of the animation and let Core Animation do the rest. You can also specify custom timing information and animation parameters as needed; however, Core Animation provides suitable default values if you do not.

<br>

在动画过程中，`Core Animation`使用硬件你做每一帧的绘制。所有你需要做的是指定开始和结束点，让`Core Animation`做剩余的内容。你也可以按需制定动画时间，和动画参数（`Core Animation`会提供适合的默认值）。

<br>

>For more information about how to initiate animations and configure animation parameters, see [Animating Layer Content](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/CoreAnimation_guide/CreatingBasicAnimations/CreatingBasicAnimations.html#//apple_ref/doc/uid/TP40004514-CH3-SW1).

<br>

关于更多的初始化动画和配置动画参数，详见[Animating Layer Content](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/CoreAnimation_guide/CreatingBasicAnimations/CreatingBasicAnimations.html#//apple_ref/doc/uid/TP40004514-CH3-SW1)

<br>

# Layer Objects Define Their Own Geometry

> layer对象有自己的`几何定义`

-------

> One of the jobs of a layer is to manage the visual geometry for its content. The visual geometry encompasses information about the bounds of that content, its position on the screen, and whether the layer has been rotated, scaled, or transformed in any way. Like a view, a layer has frame and bounds rectangles that you can use to position the layer and its content. Layers also have other properties that views do not have, such as an anchor point, which defines the point around which manipulations occur. The way you specify some aspects of layer geometry also differs from how you specify that information for a view.

<br>

layer的作用包括管理其内容的`几何视觉效果`。这种`几何视觉效果`信息包括内容的`边界`，在屏幕上的`位置`，还有layer是否有`旋转`，`缩放`，`仿射转换`。和视图一样，layer有`frame`和`bounds`矩形，你可以用来放置这个layer以及他的内容。layer也有视图没有的其他`属性`，如`锚点（anchor point）`，这是一个和动画操作相关的点。你用来描述layer的几何信息的方式也与view中有所不同。
<br>

## Layers Use Two Types of Coordinate Systems

> layer有两种坐标系统

----------

> Layers make use of both ***point-based coordinate systems*** and ***unit coordinate systems*** to specify the placement of content. Which coordinate system is used depends on the type of information being conveyed. Point-based coordinates are used when specifying values that map directly to screen coordinates or must be specified relative to another layer, such as for the layer’s ***position*** property. Unit coordinates are used when the value should not be tied to screen coordinates because it is relative to some other value. For example, the layer’s [anchorPoint](https://developer.apple.com/documentation/quartzcore/calayer/1410817-anchorpoint) property specifies a point relative to the bounds of the layer itself, which can change.

<br>

layer同时使用***基于点的坐标系统***和***单元坐标系统***来描述内容的防止。使用哪个坐标系统依赖于使用哪类信息表达位置信息。***基于点的坐标系统***被用于描述*直接映射到屏幕坐标的参数*或者是用于*描述相对于另外一个layer*，如，layer的***position***属性。单元坐标则是当参数不能被绑定到屏幕坐标，此时它是相对于另外一个参数。如，layer的[锚点](https://developer.apple.com/documentation/quartzcore/calayer/1410817-anchorpoint)属性描述了一个相对layer自身的`bounds`的点（可改变）。

<br>

> Among the most common uses for point-based coordinates is to specify the size and position of the layer, which you do using the layer’s `bounds` and `position` properties. The `bounds` defines the coordinate system of the layer itself and encompasses the layer’s size on the screen. The `position` property defines the location of the layer relative to its parent’s coordinate system. Although layers have a `frame` property, that property is actually derived from the values in the `bounds` and `position` properties and is used less frequently.

<br>

基于点的坐标做常见的作用是，描述layer`尺寸`和`位置`，通过`bounds`和`position`属性。`bounds`定义了layer自身的坐标系统，并且包含了layer的`尺寸`信息。`position`定义了layer相对于父坐标系统的位置。虽然layer有`frame`属性，但是这个属性实际上根据`bounds`和`position`而来，并且很少使用。


<br>


> The orientation of a layer’s bounds and frame rectangles always matches the default orientation of the underlying platform. Figure 1-3 shows the default orientations of the bounds rectangle on both iOS and OS X. In iOS, the origin of the bounds rectangle is in the top-left corner of the layer by default, and in OS X it is in the bottom-left corner. If you share Core Animation code between iOS and OS X versions of your app, you must take such differences into consideration.

<br>

layer的`bounds`和`frame`矩形的方向总是和`底层平台`的默认方向匹配。图1-3展示了iOS和OS X上bounds的默认方向矩形。在iOS `bounds` 的 `origin` 是左上角。在OS X上是左下角。所以在两个平台共用`Core Animation Code` ，必须要注意这点不同。

<br>

**图1-3**

![img]({{ site.img_url }}imgs/iOS/layer_coords_bounds_2x.png)

<br>

> One thing to note in Figure 1-3 is that the position property is located in the middle of the layer. That property is one of several whose definition changes based on the value in the layer’s anchorPoint property. The anchor point represents the point from which certain coordinates originate and is described in more detail in Anchor Points Affect Geometric Manipulations.

> The anchor point is one of several properties that you specify using the unit coordinate system. Core Animation uses unit coordinates to represent properties whose values might change when the layer’s size changes. You can think of the unit coordinates as specifying a percentage of the total possible value. Every coordinate in the unit coordinate space has a range of 0.0 to 1.0. For example, along the x-axis, the left edge is at the coordinate 0.0 and the right edge is at the coordinate 1.0. Along the y-axis, the orientation of unit coordinate values changes depending on the platform, as shown in Figure 1-4.

**图1-4**

![img]({{ site.img_url }}imgs/iOS/layer_coords_unit_2x.png)

> **Note:** Until OS X 10.8, the geometryFlipped property was a way to change the default orientation of a layer’s y-axis when needed. Use of this property was sometimes necessary to correct the orientation of a layer when flip transforms were in involved. For example, if a parent view used a flip transform, the contents of its child views (and their corresponding layers) would often be inverted. In such cases, setting the geometryFlipped property of the child layers to YES was an easy way to correct the problem. In OS X 10.8 and later, AppKit manages this property for you and you should not modify it. For iOS apps, it is recommended that you do not use the geometryFlipped property at all.

> All coordinate values, whether they are points or unit coordinates are specified as floating-point numbers. The use of floating-point numbers allows you to specify precise locations that might fall between normal coordinate values. The use of floating-point values is convenient, especially during printing or when drawing to a Retina display where one point might be represented by multiple pixels. Floating-point values allow you to ignore the underlying device resolution and just specify values at the precision you need.

## Anchor Points Affect Geometric Manipulations

> Geometry related manipulations of a layer occur relative to that layer’s anchor point, which you can access using the layer’s anchorPoint property. The impact of the anchor point is most noticeable when manipulating the position or transform properties of the layer. The position property is always specified relative to the layer’s anchor point, and any transformations you apply to the layer occur relative to the anchor point as well.

> Figure 1-5 demonstrates how changing the anchor point from its default value to a different value affects the position property of a layer. Even though the layer has not moved within its parents’ bounds, moving the anchor point from the center of the layer to the layer’s bounds origin changes the value in the position property.

**图1-5**

![img]({{ site.img_url }}imgs/iOS/layer_coords_anchorpoint_position_2x.png)

> Figure 1-6 shows how changing the anchor point affects transforms applied to the layer. When you apply a rotation transform to the layer, the rotations occur around the anchor point. Because the anchor point is set to the middle of the layer by default, this normally creates the kind of rotation behavior that you would expect. However, if you change the anchor point, the results of the rotation are different.

**图1-6**

![img]({{ site.img_url }}imgs/iOS/layer_coords_anchorpoint_transform_2x.png)

## Layers Can Be Manipulated in Three Dimensions

> Every layer has two transform matrices that you can use to manipulate the layer and its contents. The transform property of CALayer specifies the transforms that you want to apply both to the layer and its embedded sublayers. Normally you use this property when you want to modify the layer itself. For example, you might use that property to scale or rotate the layer or change its position temporarily. The sublayerTransform property defines additional transformations that apply only to the sublayers and is used most commonly to add a perspective visual effect to the contents of a scene.

> Transforms work by multiplying coordinate values through a matrix of numbers to get new coordinates that represent the transformed versions of the original points. Because Core Animation values can be specified in three dimensions, each coordinate point has four values that must be multiplied through a four-by-four matrix, as shown in Figure 1-7. In Core Animation, the transform in the figure is represented by the CATransform3D type. Fortunately, you do not have to modify the fields of this structure directly to perform standard transformations. Core Animation provides a comprehensive set of functions for creating scale, translation, and rotation matrices and for doing matrix comparisons. In addition to manipulating transforms using functions, Core Animation extends key-value coding support to allow you to modify a transform using key paths. For a list of key paths you can modify, see CATransform3D Key Paths.

**图1-7**

![img]({{ site.img_url }}imgs/iOS/transform_basic_math_2x.png)



> Figure 1-8 shows the matrix configurations for some of the more common transformations you can make. Multiplying any coordinate by the identity transform returns the exact same coordinate. For other transformations, how the coordinate is modified depends entirely on which matrix components you change. For example, to translate along the x-axis only, you would supply a nonzero value for the tx component of the translation matrix and leave the ty and tz values to 0. For rotations, you would provide the appropriate sine and cosine values of the target rotation angle.

**图1-8**

![img]({{ site.img_url }}imgs/iOS/transform_manipulations_2x.png)

> For information about the functions you use to create and manipulate transforms, see Core Animation Function Reference.


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


> Each set of layer objects is organized into a hierarchical structure like the views in your app. In fact, for an app that enables layers for all of its views, the initial structure of each tree matches the structure of the view hierarchy exactly. However, an app can add additional layer objects—that is, layers not associated with a view—into the layer hierarchy as needed. You might do this in situations to optimize your app’s performance for content that does not require all the overhead of a view. Figure 1-9 shows the breakdown of layers found in a simple iOS app. The window in the example contains a content view, which itself contains a button view and two standalone layer objects. Each view has a corresponding layer object that forms part of the layer hierarchy.

**图1-9**
![img]({{ site.img_url }}imgs/iOS/sublayer_hierarchy_2x.png)

> For every object in the layer tree, there is a matching object in the presentation and render trees, as shown in Figure 1-10. As was previously mentioned, apps primarily work with objects in the layer tree but may at times access objects in the presentation tree. Specifically, accessing the presentationLayer property of an object in the layer tree returns the corresponding object in the presentation tree. You might want to access that object to read the current value of a property that is in the middle of an animation.

**图1-10**
![img]({{ site.img_url }}imgs/iOS/sublayer_hierarchies_2x.png)

> **Important:** You should access objects in the presentation tree only while an animation is in flight. While an animation is in progress, the presentation tree contains the layer values as they appear onscreen at that instant. This behavior differs from the layer tree, which always reflects the last value set by your code and is equivalent to the final state of the animation.


# The Relationship Between Layers and Views


----------


> Layers are not a replacement for your app’s views—that is, you cannot create a visual interface based solely on layer objects. Layers provide infrastructure for your views. Specifically, layers make it easier and more efficient to draw and animate the contents of views and maintain high frame rates while doing so. However, there are many things that layers do not do. Layers do not handle events, draw content, participate in the responder chain, or do many other things. For this reason, every app must still have one or more views to handle those kinds of interactions.

<br>

layer不是view的替代，你不能单独的使用layer来构建你视图接口。layer为view提供`基础设施`。特别地，layer使绘制和动画视图的内容更简单高效，并且维持在高帧数。然而，也有一些事情layer并不会做，layer不会处理`事件`，`绘制内容`，`参与响应链`，以及其他。因此app必须任然有一个或者更多视图来处理这类交互。

<br>

> In iOS, every view is backed by a corresponding layer object but in OS X you must decide which views should have layers. In OS X v10.8 and later, it probably makes sense to add layers to all of your views. However, you are not required to do so and can still disable layers in cases where the overhead is unwarranted and unneeded. Layers do increase your app’s memory overhead somewhat but their benefits often outweigh the disadvantage, so it is always best to test the performance of your app before disabling layer support.

<br>

在iOS中，每个视图都有一个对应的layer，但是在OSX, 你必须自己决定那些视图需要有layer。在OS Xv10.8 或者更高，系统可能会自主的为你的视图加上layer。然而，并不是一定要这么做，你可以关掉layer当存在不合理不必要的负荷。layer 会提高你的app的内存负荷，但总体来说收益大于弊端，所以最好先测试下性能再决定是否关闭layer支持。

<br>

> When you enable layer support for a view, you create what is referred to as a ***layer-backed view***. In a layer-backed view, the system is responsible for creating the underlying layer object and for keeping that layer in sync with the view. All iOS views are layer-backed and most views in OS X are as well. However, in OS X, you can also create a ***layer-hosting view***, which is a view where you supply the layer object yourself. For a layer-hosting view, AppKit takes a hands off approach with managing the layer and does not modify it in response to view changes

<br>

当你对视图启用layer支持，你创建的视图可以称为***层次支持的视图***。对一个***层次支持视图***，系统会负责创建底层layer对象，并保持这个layer和view同步。所有的iOS视图是***层次支持视图***，大部分OS X视图也是如此。然而，在OS X,你也可以创建***层次托管视图***， 这种视图的layer是由你自己提供的。对于一个***层次托管视图***， AppKit 适宜的放权对于这个layer的管理，并且不会响应视图的变化自动做出更改。

<br>

> **Note:** For layer-backed views, it is recommended that you manipulate the view, rather than its layer, whenever possible. In iOS, views are just a thin wrapper around layer objects, so any manipulations you make to the layer usually work just fine. But there are cases in both iOS and OS X where manipulating the layer instead of the view might not yield the desired results. Wherever possible, this document points out those pitfalls and tries to provide ways to help you work around them.

<br>

**注意：**对于`layer支持视图`，尽可能最好操作视图而不是layer。在iOS上，视图知识对于layer对象的轻量包装，所以任何对于layer的操作`通常`会正常。但是有许多情况下，iOS和OS X上操作layer而不是视图可能会产生不可预期的结果。尽可能的，这个文档会列出这些隐患并且提供这些情况下的处理方案。

<br>

> In addition to the layers associated with your views, you can also create layer objects that do not have a corresponding view. You can embed these standalone layer objects inside of any other layer object in your app, including those that are associated with a view. You typically use standalone layer objects as part of a specific optimization path. For example, if you wanted to use the same image in multiple places, you could load the image once and associate it with multiple standalone layer objects and add those objects to the layer tree. Each layer then refers to the source image rather than trying to create its own copy of that image in memory.

<br>

除了关联着视图的layer，你也可以创建layer没有对应的视图对象。你可以将这些单独的layer对象嵌入到app中其他layer对象（包括关联着视图的对象）。如，你可能要创建同一张图片在许多地方，你可以加载图片一次，然后将这张图片关联到多个单独的layer对象，然后将这些对象加入到`layer tree`，每个layer引用着同一个原图而不是创建自己的内存备份中。

<br>

> For information about how to enable layer support for your app’s views, see [Enabling Core Animation Support in Your App](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/CoreAnimation_guide/SettingUpLayerObjects/SettingUpLayerObjects.html#//apple_ref/doc/uid/TP40004514-CH13-SW5). For information on how to create a layer object hierarchy, and for tips on when you might do so, see [Building a Layer Hierarchy](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/CoreAnimation_guide/BuildingaLayerHierarchy/BuildingaLayerHierarchy.html#//apple_ref/doc/uid/TP40004514-CH6-SW2).

<br>

> 更多关于开启layer支持，详见 [Enabling Core Animation Support  in Your App](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/CoreAnimation_guide/SettingUpLayerObjects/SettingUpLayerObjects.html#//apple_ref/doc/uid/TP40004514-CH13-SW5)。更多关于如何创建一个laye对象层级以及tips，详见[Building a Layer Hierarchy](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/CoreAnimation_guide/BuildingaLayerHierarchy/BuildingaLayerHierarchy.html#//apple_ref/doc/uid/TP40004514-CH6-SW2)。

<br>


# 引用
------

<!-- [CATansaction Doc][apple-doc-catransaction] -->

[Core Animation Program Guide][apple-doc-core-animation-programming-guide]

[apple-doc-catransaction]: https://developer.apple.com/documentation/quartzcore/catransaction
[apple-doc-core-animation]: https://developer.apple.com/documentation/quartzcore
[apple-doc-caanimationgroup]: https://developer.apple.com/documentation/quartzcore/caanimationgroup

[apple-doc-core-animation-programming-guide]: https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/CoreAnimation_guide/CoreAnimationBasics/CoreAnimationBasics.html


<!-- 
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
### 阴影 -->