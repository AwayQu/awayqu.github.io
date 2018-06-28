---
layout: post
title:  "Core ML"
date:   2018-03-19 22:29:22 +0800
categories: iOS
tags: [machine-learning, iOS]
---

# Core ML

> Core ML lets you integrate a broad variety of machine learning model types into your app. In addition to supporting extensive deep learning with over 30 layer types, it also supports standard models such as tree ensembles, SVMs, and generalized linear models. Because it’s built on top of low level technologies like Metal and Accelerate, Core ML seamlessly takes advantage of the CPU and GPU to provide maximum performance and efficiency. You can run machine learning models on the device so data doesn't need to leave the device to be analyzed.

> Core ML 集成了多种类型的，机器学习模型。可以额外支持30多种深度学习层级类型，同时也支持标准的，深度学习标准模型，`tree ensembles`, `SVMs`以及各种`linear models`, 由于基于低层级技术如Metal，Accelerate构建，Core ML 无缝的利用CPU以及GPU 提供最高的性能以及效率。你可以运行机器学习模型在设备上，不需要上传分析。


# Vision 

> 图像识别

You can easily build computer vision machine learning features into your app. Supported features include `face tracking, face detection, landmarks, text detection, rectangle detection, barcode detection, object tracking, and image registration.`

# NLP （Natural Language Processing）

> 自然语言

> The natural language processing APIs in Foundatin use machine learning to deeply understand text using features such as `language identification, tokenization, lemmatization, part of speech, and named entity recognition.`

![image.png]({{ site.img_url }}imgs/iOS/core-ml-01.png)


# Models

## Core ML Model

* Single document
* Public format

## Sample Models

https://developer.apple.com/machine-learning/

## Convert to Core ML 

> 支持从python的 模型 转到 Core ML 

# Development Flow

![image.png]({{ site.img_url }}imgs/iOS/core-ml-02.png)


# TODO 

待使用DEMO中的模型

# 引用

-------


[WWDC2017-Session703](https://developer.apple.com/videos/play/wwdc2017/703/)