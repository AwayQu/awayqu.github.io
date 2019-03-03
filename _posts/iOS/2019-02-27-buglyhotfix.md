---
layout: post
title:  "bugly hotfix"
date:   2019-02-27 21:53:01 +0800
categories: iOS
tags: [iOS]
---

看了下Bugly的`hotfix`（v2.1.0）的内部实现，实现基类是`BLYJCEBaseObject`以及一堆`jce`前缀的函数，用的还是`Javascript Core` 加 动态运行时实现，代码内部包含大量`patch`,`hotfix`,`objc_registerClassPair`等关键词，但是在bugly平台上能用，说明苹果的动态性检查不是特别严。

还有听别人说微信用的是虚机版实现，应该是没有把这些拿给外部用有所保留。