---
layout: post
title:  "Linear Regression & Logist Regression & NN"
date:   2018-03-29 00:35:23 +0800
categories: ML
---
* TOC
{:toc}

# Linear Regression (线性回归)

## 单特征值的线性回归

![image.png]({{ site.img_url }}imgs/ml/linear-regression.png)


## 线性回归的公式

$$
f(x_i, W, b) =  W x_i + b
$$

### 只有一个特征值时

$$
W = [\Theta_1]^T 
$$

$$
x_i = [x_1]
$$

$$
f(x_i, W, b) =  [\Theta_1]^T  [x_1] + b
$$

### 有n个特征值时

$$
W = [\Theta_1, \Theta_2, \Theta_3, \Theta_n]^T 
$$

$$
x_i = [x_1, x_2, x_3, x_n]
$$

$$
f(x_i, W, b) =  [\Theta_1, \Theta_2, \Theta_3, \Theta_n]^T   [x_1, x_2, x_3, x_n] + b
$$


### 多个特征值的线性回归1

![image.png]({{ site.img_url }}imgs/ml/mutile-linear-regression.png)


### 多个特征的线性回归2

![image.png]({{ site.img_url }}imgs/ml/cat-linear-classify.jpg)


# 逻辑回归

> 离散值的数值

![image.png]({{ site.img_url }}imgs/ml/logist-regression-1.png)


**Sigmoid function**

![image.png]({{ site.img_url }}imgs/ml/logist-regression-2.png)


# 神经网络

### 神经元(neuron)

![image.png]({{ site.img_url }}imgs/ml/neuron.png)


### 单个神经元模型 （nenuron model）

![image.png]({{ site.img_url }}imgs/ml/neuron_model.jpeg)

### 神经网络 

![image.png]({{ site.img_url }}imgs/ml/neural_net.jpeg)

### 多层神经网络

![image.png]({{ site.img_url }}imgs/ml/neural_net2.jpeg)

### 神经网络计算

![image.png]({{ site.img_url }}imgs/ml/neural_net_calculate.png)


### 卷积神经网络

> INPUT -> [[CONV -> RELU]*N -> POOL?]*M -> [FC -> RELU]*K -> FC

![image.png]({{ site.img_url }}imgs/ml/cnn.jpeg)


### 卷积神经元 连接输入

![image.png]({{ site.img_url }}imgs/ml/depthcol.jpeg)


### 卷积网络应用过程

![image.png]({{ site.img_url }}imgs/ml/convnet.jpeg)

# 引用

[linear-classify](https://cs231n.github.io/linear-classify/)

[convolutional-networks](https://cs231n.github.io/convolutional-networks/)

[Andrew Ng's Machine Learing Soursera](https://www.coursera.org/learn/machine-learning/home/welcome)
