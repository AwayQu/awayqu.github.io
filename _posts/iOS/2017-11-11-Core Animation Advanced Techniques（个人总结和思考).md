---
layout: post
title:  "Core Animation Advanced Techniques（个人总结和思考)"
date:   2017-11-11 23:38:25 +0800
categories: iOS
tags: [iOS, Core Animation]
---

# iOS渲染流程

![image.png]({{ site.img_url }}imgs/iOS/2017-11-11/渲染流程.png)


# 动画进行时

![image.png]({{ site.img_url }}imgs/iOS/2017-11-11/动画进行时.png)


<!-- 

Any layer with a mask (layer.mask)
Any layer with layer.masksToBounds / view.clipsToBounds being true
Any layer with layer.allowsGroupOpacity set to YES and layer.opacity is less than 1.0
Any layer with a drop shadow (layer.shadow*).
Any layer with layer.shouldRasterize being true
Any layer with layer.cornerRadius, layer.edgeAntialiasingMask, layer.allowsEdgeAntialiasing
Text (any kind, including UILabel, CATextLayer, Core Text, etc).
Most of the drawing you do with CGContext in drawRect:. Even an empty implementation will be rendered offscreen. 

-->

<!-- 

需要进行离屏渲染的原因

1.蒙版
2.裁剪
3.透明度 allowsGroupOpacity
4.阴影 drop shadow
5.shouldRasterize
6.圆角 反锯齿， 反锯齿蒙版
7.文本

 -->

