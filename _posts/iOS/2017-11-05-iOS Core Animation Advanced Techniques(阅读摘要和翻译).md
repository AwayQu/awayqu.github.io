---
layout: post
title:  "iOS Core Animation Advanced Techniques(阅读摘要和翻译)"
date:   2017-11-05 22:45:25 +0800
categories: iOS
tags: [iOS, Core Animation]
---

* TOC
{:toc}

# Core Animation Advanced Techniques

# The Layer Beneath

# Layer Tree

##  Layers and Views

> Core Animation is a `compositing engine`; its job is to `compose different pieces of visual content on the screen`, and to do so as fast as possible. 

Core Animation 其实是个`合成引擎`

> we’re going to cover Core Animation’s `static compositing` and `layout features`, starting with the layer tree.

`静态合成` 和 `布局特性`

> UIView `handles touch events` and supports ***Core Graphics-based drawing***, ***affine transforms*** (such as `rotation` or `scaling`), and ***simple animations*** such as `sliding` and `fading`.

UIView 处理事件， 支持 `基于Core Graphics的绘制`


> What you may not realize is that UIView does not deal with most of these tasks itself. `Rendering, layout, and animation` are all managed by a Core Animation class called `CALayer`.

`渲染，布局，动画`全都是由 ***CALayer*** 管理

> Layers, like views, are rectangular objects that can be `arranged into a hierarchical tree`. Like views, they can `contain content` (such as ***an image, text, or a background color***) and `manage the position of their children` (***sublayers***). They have `methods and properties for performing animations and transforms`. The only major feature of UIView that isn’t handled by CALayer is user interaction.

Layers 有着`层级结构`， 可以`包含内容`， 管理`子视图位置`， `用于执行动画和转换的方法和属性`


> It is actually these ***backing layers*** that are responsible for the display and animation of everything you see onscreen.

***背部视图*** 负责所有的显示和动画

> UIView is simply a wrapper, providing iOS-specific functionality such as touch handling and high-level interfaces for some of Core Animation’s low-level functionality.

`UIView 只是简单的包裹`， `提供iOS特有的功能`， 如触摸处理，对Core Animation 低层级接口的封装


> ***Why does iOS have these two parallel hierarchies based on UIView and CALayer***? Why not a single hierarchy that handles everything? The reason is to `separate responsibilities`, and `so avoid duplicating code`. ***Events and user interaction work quite differently on iOS than they do on Mac OS***; a user interface based on multiple concurrent finger touches (multitouch) is a fundamentally different paradigm to a mouse and keyboard, ***which is why iOS has UIKit and UIView and Mac OS has AppKit and NSView. They are functionally similar, but differ significantly in the implementation.***

分离职责

> Drawing, layout, and animation, in contrast, are concepts that apply just as much to touchscreen devices like the iPhone and iPad as they do to their laptop and desktop cousins. `By separating out the logic for this functionality into the standalone Core Animation framework, Apple is able to share that code between iOS and Mac OS,` making things simpler both for Apple’s own OS development teams and for third-party developers who make apps that target both platforms.

多平台共用通用逻辑

> In fact, there are not two, but ***four such hierarchies***, `each performing a different role.` In addition to the view hierarchy and layer tree, there are the presentation tree and render tree.

实际上有四层树结构`View Tree` ， `Model Layer Tree`， `Presentation Tree`， `Render Tree`


## Layer Capabilities

> But with that simplicity comes a loss of flexibility. If you want to do something slightly out of the ordinary, or make use of a feature that Apple has not chosen to expose in the UIView class interface, you have no choice but to venture down into Core Animation to explore the lower-level options.

UIView 提供的对Core Animation***接口的封装不够灵活***，有时自定义程度高需要直接动用`Core Animation`

没有暴露的功能

▪ Drop shadows, rounded corners, and colored borders ▪ 3D transforms and positioning
▪ Nonrectangular bounds
▪ Alpha masking of content
▪ Multistep, nonlinear animations

## Working with Layers

> A view has only one `backing layer (created automatically)` but can ***host*** an `unlimited number of additional layers`. 

view 只有一个`背部图层`， 但是可以代理（接待）许多额外的图层

> 
The `benefit` of using a layer-backed view instead of a hosted CALayer is that while `you still get access to all the low-level CALayer features`, `you don’t lose out on the high-level APIs` (such as **autoresizing, autolayout, and event handling**) provided by the UIView class.

同时用UIView 和 CALayer的特性

> 
You might still `want to use a hosted CALayer instead of a layer-backed UIView` in a real- world application for a few reasons, however:

* You might be writing cross-platform code that will also need to work on a Mac.
*  你可能写跨平台的代码，同时在mac上使用。
* You might be working with multiple CALayer subclasses (see Chapter6, “Specialized Layers”) and have no desire to create new UIView subclasses to host them all.
* 你可能同时使用多个CAlayer子类 不想创建一个新的视图子类去代理（接待）他们。
* You might be doing such performance-critical work that even the negligible overhead of maintaining the extra UIView object makes a measurable difference (although in that case, you’ll probably want to use something like OpenGL for your drawing anyway).
* 你可能需要实现对性能要求十分苛刻的代码。


# The backing Image

## The contents Image

> CALayer has a property called contents. This property’s type is defined as id, implying that it can be any kind of object. This is true—in the sense that you can assign any object you like to the contents property and your app will still compile—however, in practice, if you supply anything other than a CGImage, then your layer will be blank.

但是只能接受 `assign` 一个 `CGImage`对象


> This quirk of the contents property is due to Core Animation’s Mac OS heritage. The reason that contents is defined as an id is so that on Mac OS, you can assign either a CGImage or an NSImage to the property and it will work automatically. If you try to assign a UIImage on iOS, however, you’ll just get a blank layer. This is a common cause of confusion for iOS developers who are new to Core Animation.

定义成`id`的原因， 只是因为要在Mac OS上使用， Mac OS上会使用`NSImage`， （真的是缺乏静态信息的动态语音）

> The headaches don’t stop there, though. The type you actually need to supply is a CGImageRef, which is a pointer to a CGImage struct. ***UIImage has a CGImage property that returns the underlying `CGImageRef`***. If you try to assign that to the CALayer contents property directly, though, it won’t compile because CGImageRef is not really a Cocoa object; it’s a Core Foundation type.

令人头疼的事情还有， UIImage有一个CGImage对象 会返回 CGImageRef，所以不能直接赋值， 从 Core Foundation 赋予 Cocoa Object。

> Although Core Foundation types behave like Cocoa objects at runtime (known as toll-free bridging), they are not type compatible with id unless you use a bridged cast. To assign the image of a layer, you actually need to do the following:


```objectivec
layer.contents = (__bridge id)image.CGImage;
```

运行时，Core FOundation 类型的行为 和 Cocoa object 类似， 但是他们还是需要进行桥接


> That was some very simple code, and yet we’ve done something quite interesting here: Using the power of CALayer, we’ve displayed an image inside an ordinary UIView. This isn’t a UIImageView; it’s not designed to display images normally. By manipulating the layer directly, we’ve exposed new functionality and made our humble UIView a lot more interesting.

直接显示一张图片，在一个没有UIImageView的 UIView 内。

## contentsGravity （内容重心）

> You might have noticed that our snowman looks a bit... fat. The image we loaded wasn’t precisely square, but it’s been stretched to fit the view. You’ve probably seen a similar situation when using UIImageView, and the solution there would be to set the contentMode property of the view to something more appropriate, like this:

雪人看起来有点胖， 因为图片没有加载到恰好的区域

如果使用UIImageView 有如下解决方案

```objectivec
view.contentMode = UIViewContentModeScaleAspectFit;
```

> The equivalent property of CALayer is called contentsGravity, and it is an NSString rather than an enum like its UIKit counterpart. The contentsGravity string should be set to one of the following constant values:

```
kCAGravityCenter kCAGravityTop kCAGravityBottom kCAGravityLeft kCAGravityRight kCAGravityTopLeft
kCAGravityTopRight kCAGravityBottomLeft kCAGravityBottomRight kCAGravityResize kCAGravityResizeAspect kCAGravityResizeAspectFill
```

等价的属性在CALayer中叫做 contentsGravity，


## contentsScale （内容比例）


## masksToBounds （蒙版到边界）


## contentsRect  (内容矩形)

> The following coordinate types are used in iOS:

* `Points`—The most commonly used coordinate type on iOS and Mac OS. Points are virtual pixels, also known as logical pixels.
* `Pixels`—Physical pixel coordinates are not used for screen layout, but they are often still relevant when working with images.
* `Unit`—Unit coordinates are a convenient way to specify measurements that are relative to the size of an image or a layer’s bounds, and so do not need to be adjusted if that size changes.
iOS上有以下几种坐标类型：
* 点， 虚拟像素， 逻辑像素
* 像素， 物理像素
* 单元， 单元坐标是一种方便的描述尺寸的方式， 是相对于一个图片的尺寸或者是一个layer的边界，并且不需要去调整如果尺寸改变了

单元总单位是1。 0.5是一半


## contentsCenter

  ![Alt text](./Screen Shot 2017-10-26 at 23.53.41.png)



## Custom Drawing


> Setting the layer contents with a CGImage is not the only way to populate the backing image. ***It is also possible to draw directly into the backing image `using Core Graphics`.*** The -drawRect: method can be implemented in a UIView subclass to implement custom drawing.

可以使用 `Core Graphics`绘制背部图片


> The -drawRect: method has no default implementation because a ***UIView does not require a custom backing image*** *if it is just filled with a solid color* or *if the underlying layer’s contents property contains an existing image instance*. If UIView detects that the -drawRect: method is present, ***it allocates a new backing image for the view,*** with pixel dimensions equal to the view size multiplied by the contentsScale.

如果drawRect被实现了， 会分配一个新的`背部图片`给这个view，**像素尺寸  = 视图尺寸 * 内容比例**


> If you don’t need this backing image, it’s a waste of memory and CPU time to create it, which is why Apple recommends that you don’t leave an empty -drawRect: method in your layer subclasses if you don’t intend to do any custom drawing.

如果不需要`背部图片`， 建议不要留空的 `drawRect`方法


> The -drawRect: method is ***executed automatically when the view first appears onscreen.*** The code inside -drawRect: method uses Core Graphics to draw into the backing image, **the result will then be `cached` until the view needs to update it** (usually because the developer `has called the -setNeedsDisplay` method, although some view types will `be redrawn automatically whenever a property that affects their appearance is changed [such as bounds]`). Although -drawRect: is a UIView method, **it’s actually `the underlying CALayer that schedules the drawing and stores the resultant image`.**

视图第一次显示在图片上时， drawRect的方法会自动调用， drawRect中使用Core Graphics会绘制到`背部图片`上。

绘制结果会缓存被 CALayer持有， 直到被更新再次绘制1.-setNeedsDisplay 2.或者是会影响显示的属性改变 如bounds

-drawRect是 UIView的方法， 但是实际上是 `CALayer 计划安排绘制以及存储结果图片`。


> CALayer has an optional delegate property that conforms to the `CALayerDelegate` protocol. When CALayer requires content-specific information, it requests it from the delegate. CALayerDelegate is ***an informal protocol***, which is a fancy way of saying that there is no actual CALayerDelegate @protocol that you can reference in your class interface. You just add the methods you need and CALayer will call them if present. (The delegate property is just declared as an id, and `all the delegate methods are treated as optional`.)

CALayer 有一个可选的delegate property， 叫做`CALayerDelegate`， 当需要`特定内容信息`，回想这个delegate获取， 这是一个`友好的协议`， 全部是可选方法。



> When it needs to be redrawn, CALayer asks its delegate to supply a backing image for it to display. It does this by attempting to call the following method:

```objectivec
- (void)displayLayer:(CALayer *)layer;
```

当需要重新绘制， CALayer会询问他的代理去提供一个`背部图片`让其显示，


> This is an opportunity for the delegate to set the layer contents property directly if it wants to, in which case no further methods will be called. If the delegate does not implement the -displayLayer: method, CALayer attempts to call the following method instead:

```objectivec
- (void)drawLayer:(CALayer *)layer inContext:(CGContextRef)ctx;
```

存在时机，代理可以直接设置layer的内容属性，在没有更多其他方法被调用时，并且代理没有实现`-displayLayer:`，CALayer会视图调用一下方法


> Before calling this method, CALayer creates an empty backing image of a suitable size (based on the layer bounds and contentsScale) and a Core Graphics drawing context suitable for drawing into that image, which it passes as the ctx parameter.

在调用这个方法前，CALayer会创建一个空的`背部图片`，并且一个Core Graphics 绘制上下文恰当地绘制入这个`背部视图`。


>Note a couple of interesting things here:
  * We have to manually call -display on blueLayer to force it to be updated. Unlike UIView, CALayer does not redraw its contents automatically when it appears onscreen; it is left to the discretion of the developer to decide when the layer needs redrawing.
当layer已经显示后， 需要显示调用`-display`使它重新绘制
  * The circle that we have drawn is clipped to the layer bounds even though we have not enabled the masksToBounds property. That’s because when you draw the backing image using the CALayerDelegate, the CALayer creates a drawing context with the exact dimensions of the layer. There is no provision made for drawing that spills outside of those bounds.
即使没有设置`masksToBounds`, 因为`背部图片`的`绘制上下文`已经声明了一定的尺寸，没法绘制超出这个区域。

-------

> So now you understand the CALayerDelegate and how to use it. But unless you are creating standalone layers, you will almost never need to implement the CALayerDelegate protocol. The reason for this is that when UIView creates its backing layer, it automatically sets itself as the layer’s delegate and provides an implementation for -displayLayer: that abstracts these issues away.

除非创建单独的layer， 否则基本不用实现 `CALayerDelegate`， 因为UIView会自己创建他的`背部图层`，并且自动设置好他的delegate，提供`-displayLayer`的实现

> When using view-backing layers, you do not need to implement -displayLayer: or -drawLayer:inContext: to draw into your layer’s backing image; you can just implement the -drawRect: method of UIView in the normal fashion, and UIView takes care of everything, including automatically calling -display on the layer when it needs to be redrawn.

当使用视图背部图层是，你不需要实现 `-displayLayer:` or `-drawLayer:inContext:` 绘制内容到你的图层的背部iamge，你可以只是实现`-drawRect`，UIView会做所有的事情包括自动调用 `-display`



# Layer Geometry （图层几何）

> This chapter covered the geometry of CALayer, including its frame, position, and bounds, and we touched on the concept of layers existing in three-dimensional space instead of a flat plane. We also discussed how touch handing can be implemented when working with hosted layers, and the lack of support for autoresizing and autolayout in Core Animation on iOS.

图层几何， frame， position， 以及bounds， 图层的概念 存在于三维空间而不是平面， 处理触摸时间当使用委托图层，缺乏的对于autoresizing以及autolayout的支持在Core Animation中。


# Visual Effects （视觉效果）

> This chapter explored some of the visual effects that you can apply programmatically to layers, such as rounded corners, drop shadows, and masks. We also looked at `scaling filters and group opacity.`

这张显示了一些视觉效果，你可以编码应用与图层，圆角，阴影，蒙版。


# Transform （转换）

> This chapter covered 2D and 3D transforms. You learned a bit about matrix math, and how to create 3D scenes with Core Animation. You saw what the back of a layer looks like and learned that you can’t peer around objects in a flat image. Finally, this chapter demonstrated that when it comes to handling touch events, the order of views or layers in the hierarchy is more significant than their apparent order onscreen.

2D 和 3D转换， 当有事件处理时，视图的顺序，以及视图的层级顺序 比 视图的显示顺序 更加重要。


# Specialized Layers （特别的layers）

> This chapter provided an overview of the many specialized layer types and the effects that can be achieved by using them. We have only scratched the surface in many cases; classes such as CATiledLayer or CAEmitterLayer could fill a chapter on their own. However, the key point to remember is that CALayer is a jack-of-all-trades, and is not optimized for every possible drawing scenario. To get the best performance out of Core Animation, you need to choose the right tool for the job, and hopefully you have been inspired to dig deeper into the various CALayer subclasses and their capabilities.
这章讲了一些特殊的视图，以及可以通过这些视图做到的效果。但只是划破表面而已。


# Setting Things in Motion


## Implicit Animations

### Transactions

> Core Animation is built on the assumption that everything you do onscreen will (or at least may) be animated. Animation is not something that you enable in Core Animation. Animations have to be explicitly turned off; otherwise, they happen all the time

Core Animation 是被建立在假定你在屏幕上做的所有事情都需要被动画，除非显示关闭，否则动画一直在发生。 (默认有0.25s动画)

> The duration of the animation is specified by the settings for the current `transaction`, and the animation type is controlled by `layer actions`.

动画的时间 描述在 `事务`中， 动画的类型通过 `图层动作` 控制。

> Transactions are the mechanism that Core Animation uses to encapsulate a particular set of property animations. 

事务时一种概括包含一些列特定属性动画的机制

> Core Animation automatically begins a new transaction with each iteration of the run loop. (The run loop is where iOS gathers user input, handles any outstanding timer or network events, and then eventually redraws the screen.) Even if you do not explicitly begin a transaction using [CATransaction begin], any property changes that you make within a given run loop iteration will be grouped together and then animated over a 0.25-second period.

`Core Animation` 自动开启一个新的事务，在每一次迭代run loop的时候，（run loop 是iOS 汇集用户输入，处理到达的定时器，以及网络事件，然后最终重绘屏幕） 即使你不显示begin一个`事务`，任何你更改的属性，在run loop迭代中将会被组合在一起进行一个0.25秒周期的动画

> Armed with this knowledge, we can easily change the duration of our color animation. It would be sufficient to change the animation duration of the current (default) transaction by using the +setAnimationDuration: method, but we will start a new transaction first so that changing the duration doesn’t have any unexpected side effects. Changing the duration of the current transaction might possibly affect other animations that are incidentally happening at the same time (such as screen rotation), so it is always a good idea to push a new transaction explicitly before adjusting the animation settings.

由于直接改系统开启的动画duration，会影响到同时发生的其他动画，如附带的转屏动画，所以最好的做好是开启一个新的事务。

#### Completion Blocks
可以设置回调

#### Layer Actions

> The animations that CALayer automatically applies when properties are changed are called actions.

当属性改变时，CALayer自动应用的动画叫做`actions`。

**查找当前的`action`的顺序**
1. The layer first checks whether it has a delegate and if the delegate implements the -actionForLayer:forKey method specified in the CALayerDelegate protocol. If it does, it will call it and return the result.
2. If there is no delegate, or the delegate does not implement -actionForLayer:forKey, the layer checks in its actions dictionary, which contains a mapping of property names to actions.
3. If the actions dictionary does not contain an entry for the property in question, the layer searches inside its style dictionary hierarchy for any actions that match the property name.
4. Finally, if it fails to find a suitable action anywhere in the style hierarchy, the layer will fall back to calling the -defaultActionForKey: method, which defines standard actions for known properties.

#### Presentation Versus Model

> the layer tree is sometimes referred to as the `model layer tree`.

图层树有时候被叫做模型图层

> The display values of each layer’s properties are stored in a separate layer called the presentation layer, which is accessed via the `-presentationLayer` method.

每个图层，显示的数值被存在一个分离的图层叫做展示图，通过`-presentationLayer`获取

> Note that the presentation layer is only created when a layer is first ***committed*** (that is, when it’s first displayed onscreen), so attempting to call -presentationLayer before then will return nil.

展示树图层只有被第一次***提交***（展示在屏幕上）之后才会存在，在这之前访问会返回nil

**只有需要实现`基于时间的动画`以及`使动画的layer响应时间调用-hitTest:`才会用到`展示树`**
 * If you are implementing timer-based animations (see Chapter 11, “Timer-Based Animation”) in addition to ordinary transaction-based animations, it can be useful to be able to find out exactly where a given layer appears to be onscreen at a given point in time so that you can position other elements to correctly align with the animation.
 * If you want your animated layers to respond to user input, and you are using the -hitTest: method (see Chapter 3, “Layer Geometry”) to determine whether a given layer is being touched, it makes sense to call -hitTest: against the presentation layer rather than the model layer because that represents the layer’s position as the user currently sees it, not as it will be when the current animation has finished.

## Explicit Animations （显示动画）



# Tuning for Speed （调节速度）

## CPU Versus GPU

### The Stages of an Animation

* Layout   准备视图以及图层层级，以及设置layer的属性 （frame， background color， border，以及其他）
* Display  图层的backing image 绘制， 包含 `-drawRect:` 以及 `-drawLayer:inContext:`
* Prepare 准备好需要传给`render server`的数据，包括解压图片等等
* Commit 打包好图层以及动画属性， 通过进程间通信，传给渲染服务（`render server`）用于显示

> But those are just the phases that take place inside your application—there is still more work to be done before the animation appears onscreen. Once the packaged layers and animations arrive in the render server process, they are deserialized to form another layer tree called the render tree (mentioned in Chapter 1, “The Layer Tree”). Using this tree, the render server does the following for each frame of the animation:

一旦打包的layers 以及 动画  数据到达 渲染服务进程（redner server process）, 他们会被反序列化到另外一种 图层树， 叫做渲染树 （render tree）， 渲染服务会生成动画的每一帧

* **`Calculates the intermediate values for all the layer properties and sets up the OpenGL geometry (textured triangles) to perform the rendering`**


计算所有layer的中间值，并转化到OpenGL几何（纹理三角）来执行渲染

* Renders the visible triangles to the screen

渲染可视三角到屏幕

> So that’s six phases in total; the last two are repeated over and over for the duration of the animation. The first five of these phases are handled in software (by the CPU), only the last is handled by the GPU. Furthermore, you only really have direct control of the first two phases: layout and display. The Core Animation framework handles the rest internally and you have no control over it.

总共有六步， 最后两步会在动画进行中时，不断重复， 第五步是被软件执行（CPU），只有最后一步是GPU处理。而且，你只能直接控制前两步（Layout， display）， Core Animation内部会处理剩余的所有。


### GPU-Bound Operations

* Too much geometry （太多几何元素）
* Too much overdraw （太多重绘）
* Offscreen drawing （离屏绘制）
* Too-large images （太大的图片）

### CPU-Bound Operations
* Layout calculations （布局计算）
* Lazy view loading （视图懒加载，包括viewController 视图第一次显示的加载， 以及通过nib加载属兔）
* CoreGraphicsdrawing （通过`-drawRect：`以及`-drawLayer：inContext：`方法绘制图像）
* Image decompression （图片解压）

### IO-Bound Operations

加载本地文件，下载网络文件


### Core Animation 调试
* Color Blended Layers （颜色混合图层）
* Color Hits Green and Misses Red （缓存命中绿色，未命中红色） 
`shouldRasterize`：default false. A Boolean that indicates whether the layer is rendered as a bitmap before compositing. Animatable（一个布尔指示是否需要渲染成一张位图，在组合前）
* Color Copied Images 有时`backing image`创建表示Core Animation强制创建了图片的拷贝，并且将它传给渲染服务`render server`， 图片拷贝是非常消耗cpu的。
* Color Immediately  图层debug颜色，只10微秒更新一次，有些效果的调试上，这样太慢去发现问题，开启后会每一帧都去更新debug颜色，这样会影响到真实的帧率
* Color Misaligned Images 高亮未对齐的图片 （不是整数的坐标）有时候偶然，比如给一张大图显示缩略图，或者图像模糊的时候，这会很有用。
* Color Offscreen-Rendered Yellow 需要离屏渲染的视图标记为黄色， 如使用了`shadowPath` or `shouldRasterize`等属性.
* Flash Updated Regions  高亮进行了重新绘制的区域 （使用了Core Graphics 进行了软件绘制的）

### 目前理解的整体流程

CPU计算布局，加载视图，解压图片，绘制2D图形后，将图层打包传输给渲染模型，遍历渲染树（render tree）将图层（layer）转化为纹理三角，

交由GPU进行转化合成显示到屏幕上。

## Layer Performance

### Inexplicit Drawing (含糊不清的绘制)

> The layer backing image can be drawn on-the-fly using Core Graphics, or set directly using the contents property by supplying an image loaded from a file, or drawn beforehand in an offscreen CGContext. In the previous two chapters, we talked about optimizing both of these scenarios. But in addition to explicitly creating a backing image, you can also create one implicitly through the use of certain layer properties, or by using particular view or layer subclasses.
It is important to understand exactly when and why this happens so that you can avoid accidentally introducing software drawing if it’s not needed.

layer的`背部图片`可以使匆匆忙忙的使用Core Graphics绘制，或者是直接设置contents属性，或者是事先在一个离屏的CGContext中绘制。除了显示创建一个`背部图片`外，你也可以隐式创建通过特定的layer属性，或者是使用特定的视图或者是特定的layer子类。

### Text （文本）

> Both CATextLayer and UILabel draw their text directly into the backing image of the layer. These two classes actually use radically different approaches for rendering text: In iOS 6 and earlier, UILabel uses WebKit’s HTML rendering engine to draw its text, whereas CATextLayer uses Core Text. The latter is faster and should be used preferentially for any cases where you need to draw a lot of text, but they both require software drawing and are therefore inherently slow compared to hardware-accelerated compositing.

`CATextLayer` 和 `UILabel` 都可以用于将文本直接转为`背部图片`，但是这两个类其实使用不同的方式来渲染文本，在iOS6或者更早，UILabel使用 webKit的HTML渲染引擎来绘制文本， CATextLayer使用Core Text，后者会比前者更快，并且在绘制大量文本时，更应该使用。**但是两者都是使用`软件绘制`**，所以内在的比起**硬件加速的合成**慢一些。

> Wherever possible, try to avoid making changes to the frame of a view that contains text, because it will cause the text to be redrawn. For example, if you need to display a static block of text in the corner of a layer that frequently changes size, put the text in a sublayer instead.

尽可能，避免改变含有文本的视图的frame，这会导致文本的**重绘**。如，当你需要显示一个一块静态的文本在一个经常要改变尺寸的layer中，最好将其放在一个sublayer中。

### Rasterization （栅格化）

> We mentioned the `shouldRasterize` property of CALayer in Chapter 4, “Visual Effects,” as a way to solve blending glitches with overlapping translucent layers, and again in Chapter 12, “Tuning for Speed,” as a performance optimization technique when drawing complex layer subtrees.

我们提到`shouldRasterize`属性在第四章视觉效果，作为解决部分重叠的半透明图层的混合失灵的一种方式，在第12章调节速度中，作为一种性能优化技术党绘制复杂的图层树时。

> Enabling the `shouldRasterize` property causes the layer to be drawn into an offscreen image. That image will then be cached and drawn in place of the actual layer’s contents and sublayers. If there are a lot of sublayers or they have complex effects applied, this is generally less expensive than redrawing everything every frame. But it takes time to generate that rasterized image initially, and it will consume additional memory.

启动`shouldRasterize`属性会造成图层绘制成一张离屏的图片，这张图片会被缓存，并且绘制入真实的layer的内容以及sublayer中。如果存在许多sublayers，或者复杂的效果被应用，这是普遍地比每一帧都重绘制更节省。但是为了初始化生成栅格化的图片，将会消耗额外的内存。


###  Offscreen Rendering

> Offscreen rendering is invoked whenever the combination of layer properties that have been specified mean that the layer cannot be drawn directly to the screen without pre- compositing. Offscreen rendering does not necessarily imply software drawing, but it means that the layer must first be rendered (either by the CPU or GPU) into an offscreen context before being displayed. The layer attributes that trigger offscreen rendering are as follows:
  * Rounded corners(when combined with `masksToBounds`)  
  * Layer masks
  * Drop shadows

离屏渲染被唤起当**混合图层**的属性被声明，并且意味着layer不能被直接绘制在屏幕内如果没有`预合成`的话，离屏渲染并不是必须使用`软件绘制`，但是意味着这个图层必须第一次被渲染入`离屏上下文`，在显示之前，如下的属性会触发离屏渲染

> Offscreen rendering is similar to what happens when we enable rasterization, except that the drawing is not normally as expensive as rasterizing a layer, the sublayers are not affected, and the result is not cached, so there is no long-term memory hit as a result. Too many layers being rendered offscreen will impact performance significantly, however.

离屏渲染与栅格化时发生的事情相似，除了这种绘制并不像栅格化那样同样代价昂贵。**sublayer并不会受到影响**，**并且这个结果并不会被缓存**，所以并不会有长期的内存占用。虽然太多的图层被离屏渲染会显著地影响性能。

> It can sometimes be beneficial to enable rasterization as an optimization for layers that require offscreen rendering, but only if the layer/sublayers do not need to be redrawn frequently.

有时候开启栅格化来离屏渲染是非常有益的，但是只有这个layer以及sublayer不需要被经常重新绘制。

> For layers that require offscreen rendering and need to animate (or which have animated sublayers), you may be able to use CAShapeLayer, contentsCenter, or shadowPath to achieve a similar appearance with less of a performance impact.

对于一个需要离屏渲染以及需要动画的图层，你可以使用`CAShapeLayer`，`contentsCenter`，或者`shadowPath`来完成相同显示效果，但是对性能影响更小。

####  CAShapeLayer

> Neither `cornerRadius` nor `masksToBounds` impose any significant overhead on their own, but when combined, they trigger offscreen rendering. You may sometimes find that you want to display rounded corners and clip sublayers to the layer bounds, but you don’t necessarily need to clip to the rounded corners, in which case you can avoid this overhead by using CAShapeLayer.

 `cornerRadius` 或者 `masksToBounds`  都会强制造成显著的超负荷，但是将两者组合，他们将会触发`离屏渲染`，有时候，你想显示圆角以及裁剪sublayer到边界，但是你不不需要裁剪到圆角，这种时候你可以通过使用`CAShapeLayer`来避免超负荷。

> You can get the effect of rounded corners and still clip to the (rectangular) bounds of the layer without incurring a performance overhead by drawing the rounded rectangle using the handy +bezierPathWithRoundedRect:cornerRadius: constructor for UIBezierPath (see Listing 15.1). This is no faster than using cornerRadius in itself, but means that the masksToBounds property no longer incurs a performance penalty.

你可以获取到圆角效果并且任然裁剪边界，并且不产生由于绘制圆角产生的性能过载通过使用`+bezierPathWithRoundedRect:cornerRadius: `来构造`UIBezierPath`，这并不比使用圆角自身快，但是`masksToBoudns`不会再导致性能问题。

#### Stretchable Images
> Another way to create a rounded rectangle is by using a circular contents image combined with the contentsCenter property mentioned in Chapter 2, “The Backing Image,” to create a stretchable image (see Listing 15.2). In theory, this should be slightly faster to render than using a CAShapeLayer because drawing a stretchable image only requires 18 triangles (a stretchable image is rendered using nine rectangles arranged in a 3x3 grid), whereas many more are needed to render a smooth curve. In practice, the difference is unlikely to be significant.


另外一种创造一个圆角的矩形的方式是，使用一个圆形内容的图片使用`contentsCenter`属性。理论上，这种方式更加轻量快速比起`CAShapeLayer`，因为一个伸缩图片只需要18个三角（一张伸缩图片是使用一个3x3的矩形），然而需要更多去显示一个顺滑流畅的曲线，所有在实践中，这种区别并不显著。

#### shadowPath
> We mentioned the shadowPath property in Chapter 2. If your layer is a simple geometric shape like a rectangle or rounded rectangle (which it will be if it doesn’t contain any transparent parts or sublayers), it is easy to create a shadow path that matches its shape and this will greatly simplify the calculations that Core Animation has to do to draw the shadow, avoiding the need to precompose the layer offscreen. This makes a huge difference to the performance.


如果你的图层是一个简单的几何图形，如矩形或者圆角矩形（如果没有含有透明部分以及sublayers），这是很容易创建一个匹配的阴影路径（shawdowPath），并且对于Core Animation而言绘制这个阴影的计算很简单，避免这种离屏预合成将会节省很多的性能。

> If your layer has a more complex shape, it might be awkward to generate the correct shadow path, in which case you may want to consider pregenerating your shadow as a background image using a paint program.
如果你的图层是一个复杂的形状，或许会难以生成正确的shadowPath， 这种情况, 你需要考虑生成你的shadow成为一张背景图，通过使用`绘制程序`。


###  Blending and Overdraw （混合和重绘）

> As mentioned in Chapter 12, there is a limit to the number of pixels that the GPU can draw each frame (known as the ***fill rate***), and while it can comfortably draw an entire screen full of pixels, it might begin to lag if it needs to keep repainting the same area multiple times due to overlapping layers (overdraw).

GPU可在每帧绘制的像素点数量是有限制的（***填充率***），虽然可以舒适的绘制一整个屏幕的像素，但是还是有可能有耽搁延迟如果需要重复画一个相同的区域多次，因为部分重叠的图层（overdraw）。

> The GPU will discard pixels in layers that are fully obscured by another layer, but calculating whether a layer is obscured can be complicated and processor intensive. Merging together the colors from several overlapping translucent pixels (blending) is also expensive. You can help to speed up the process by ensuring that layers do not make use of transparency unless they need to. Whenever possible, you should do the following:
  * Set the background Color of your view to a fixed, opaque color.   
  * Set the opaque property of the view to YES.

GPU会丢弃一部分layer的像素（被另外一个layer完全遮盖），但是会计算一个layer是否被遮盖是`复杂` `计算密集`的。合并多个重叠的透明像素的颜色（混合）是非常消耗资源的。你可以加速这个处理通过确认这个图层不需要有透明除非真的有必要。尽可能做到下面两步：
* 设置背景色为固定的不透明颜色
* 设置opaque属性为YES

> This reduces blending (because the compositor knows that nothing behind the layer will contribute to the eventual pixel color) and speeds up the calculations for avoiding overdraw because Core Animation can discard any completely obscured layers in their entirety instead of having to test each overlapping pixel individually.

这将会减少混合（因为**合成者**知道在这个图层后面没有什么会对最终的像素颜色造成影响），并且会加速避免重绘的计算，因为Core Animation 知道完全丢弃整个被遮盖的视图，而不是去单独测试每个重叠的像素。

> If you are using images, try to avoid alpha transparency unless it is strictly needed. If the image will appear in front of a fixed background color, or a static background image that doesn’t need to move relative to the foreground, you can prefill the image background and avoid runtime blending.

如果你使用图片，要避免alpha透明。如果这个图片将会显示在一个固定的颜色在一个固定的背景色前，或者是一个静态的背景图片，不需要移到前台，**你可以在后台预先填充这张背景图片，避免运行时混合。**

> If you are using text, a UILabel with a white background (or any other solid color) is more efficient to draw than one with a transparent background.

如果你使用文本，一个`UILabel`有着固定颜色，会比有透明背景的更快。

> Finally, by judicious use of the shouldRasterize property, you can collapse a static layer hierarchy into a single image that doesn’t need to be recomposited each frame, avoiding any performance penalties due to blending and overdraw between the sublayers.

最后，谨慎使用`shouldRAsterize`属性，你可以收起一个静态的图层层级成为一张单独的图片，这样就不需要每一帧重新合成，避免子图层进行`混合`和`重绘`造成的性能问题。

### Reducing Layer Count

> Due to the overhead of allocating layers, preprocessing them, packaging them up to be sent over IPC to the render server, and then converting them to OpenGL geometry, there is a practical upper limit on the number of layers you can display onscreen at once.

由于分配图层，预处理图层，打包图层并通过IPC传输给`render server`以及图层转化为OpenGL几何（纹理三角）的消耗，**所以可以显示在屏幕上的layer有明确的上限。**

> The exact limit will depend on the iOS device, the type of layer, and the layer contents and properties, but in general once the number of layers runs into the hundreds or thousands, you are going to start to see performance problems even if the layers themselves are not doing anything particularly expensive.

这个准确的上限依赖于`iOS 设备`，`layer类型`，`layer内容和属性`，但是普遍地，一旦layer的数量到达成百上千，你将会看到明显的性能问题，即时这些layer自身没有特别明显的性能消耗。


#### Clipping
> Before doing any other kind of optimization to your layers, the first thing to check is that you are not creating and attaching layers to the window if they will not be visible. Layers might be invisible for a variety of reasons, such as the following:
  * They lie outside of the bounds of the screen, or the bounds of their parent layer.   
  * They are completely obscured by another opaque layer. 
  *  They are fully transparent.

在做任何优化前，最早需要确认的事情是你没有创建并且附上不会被显示的图层。图层可能不可见有以下几种原因：
* 他们在屏幕边界外，或者父图层的边界外。
* 他们完全被另外一个不透明的图层遮盖。
* 他们完全透明

>  Core Animation does a fairly good job of culling layers that aren’t going to contribute to the visible scene, but your code can usually determine if a layer isn’t going to be needed earlier than Core Animation can. Ideally, you want to determine this before the layer object is ever created, to avoid the overhead of creating and configuring the layer unnecessarily.

Core Animation 有相当不错机制（去除不贡献显示效果的图层），但是你的代码比Core Animation可以更早确认一个图层是否需要。理想状态下，不需要显示的图层不要被创建。

#### Core Graphics Drawing

> After you have eliminated views or layers that are not contributing to the display onscreen, there might still be ways that you can reduce the layer count further. For example, if you are using multiple UILabel or UIImageView instances to display static content, you can potentially replace it all with a single view that uses -drawRect: to replicate the appearance of a complex view hierarchy.

在你排除不需要显示在屏幕上的视图以及图层之后，你任然有可能更进一步减少图层数量。如：如果你使用多个UILabel或者UIImageView实例去显示静态的内容，你可以替换所有的这些内容为一个单独的视图通过使用`-drawRect:` 来替代一个复杂的视图层级。

> It might seem counterintuitive to do this because we know that software drawing is slower than GPU compositing and requires additional memory, but in a situation where the performance is limited by the number of layers, software drawing may actually improve the performance by avoiding excessive layer allocation and manipulation.

你也许会发现适得其反，因为软件绘制比GPU合成更慢，并且需要额外的内存，但是某种情况下由于layer的数量造成的性能问题，使用软件绘制可以改善提高通过避免昂贵的图层分配以及图层操作。

> Doing the drawing yourself in this case involves a similar performance tradeoff to rasterizing, but means that you can remove sublayers from the layer tree altogether (as opposed to just obscuring them, as you do when using shouldRasterize).

自己绘制和栅格化有相似的性能问题，需要权衡，但是可以移除所有的子图层，和栅格化隐藏子图层一样。


### The -renderInContext: Method
> Using Core Graphics to draw a static layout may sometimes be faster than using a hierarchy of UIView instances, but using UIView instances is both more concise and more flexible than writing the equivalent drawing code by hand, especially if you use Interface Builder to do the layout. It would be a shame to have to sacrifice those benefits for the sake of performance tuning.

使用Core Graphics 绘制静态布局，优势会比使用视图层级快，但是`UIView`更加的简介，并且更加的灵活，为了性能调优而放弃这些`优势`有些遗憾。

> Fortunately, you don’t have to. Having a large number of views or layers is only a problem if the layers are actually attached to the screen. Layers that are not connected to the layer tree don’t get sent to the render server and won’t impact performance (after they’ve been initially created and configured).

幸运的是，你不用必须这么做。有大量的视图以及图层会成为问题，只有将图层确实的附在屏幕上时。没有连接如图层树的图层不传输给渲染服务，不会造成性能问题。

> By using the CALayer -renderInContext: method, you can draw a snapshot of a layer and its sublayers into a Core Graphics context and capture the result as an image, which can then be displayed directly inside a UIImageView, or as the contents of another CALayer. Unlike using shouldRasterize—which still requires that the layers be attached to the layer tree—with this approach there is no ongoing performance cost.

通过使用`CALayer`的`-renderInContext：`方法，你可以绘制一张图层以及子图层的快照到一个Core Graphics Context，并且抓取结果成为一张图片，直接显示到`UIIamgeView`中，或者作为另外一个`CALayer`的`contents`。不象使用`shouldRasterize`，会继续要求图层被附在图层树上，这种方式没有持续的性能消耗。

> Responsibility for refreshing this image when the layer content changes would be up to you (unlike using shouldRasterize, which handles caching and cache invalidation automatically), but once the image has initially been generated, you save significant per- frame performance overhead with this approach versus asking Core Animation to maintain a complex layer tree.

更换缓存的责任在你，不象`shouldRasterize`完全由系统控制，为每一帧都节省了性能消耗相对于让Core Animation维护一个复杂的layer tree

# 名称解释

### overdraw （重绘）

重新绘制

### texture triangles （纹理三角）


### rasterize（栅格化)

像素化


### speculative loading (预先加载)


### bitmap （位图）

点阵图像或绘制图像，是由称作像素（图片元素）的单个点组成的。

### offscreen render 离屏渲染


### BackBoard

iOS6 之后的render sever （渲染服务进程） ， 用于动画以及组合屏幕上的图层

### IPC (Inter-Process Communication) 

进程间通信

### premature optimization

过早优化


### fill rate （填充率）

填充率