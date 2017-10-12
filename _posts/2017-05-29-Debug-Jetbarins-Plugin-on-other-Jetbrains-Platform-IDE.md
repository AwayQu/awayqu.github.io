---
layout: post
title:  "Debug Jetbarins Plugin on other Jetbrains Platform IDE (在不同IDE中Debug Jetbrains Plugin)"
date:   2017-05-29 02:32:22 +0800
categories: jetbrains
---

### 1.  打开 Project Structure (⌘ + ;)

![image.png]({{ site.img_url }}imgs/jetbrains/plugin-debug-00.png)

### 2. 新建一个SDK

![image.png]({{ site.img_url }}imgs/jetbrains/plugin-debug-01.png)

### 3. 使用( ⌘ +  ⇧ + G) 跳转对应IDE的contents目录选择确定

![image.png]({{ site.img_url }}imgs/jetbrains/plugin-debug-02.png)

#### 4.index 结束后会有提示对应的IDE中缺少对应的Jetbrains Platform SDK (直接点<u>Attach annotations</u>)
![image.png]({{ site.img_url }}imgs/jetbrains/plugin-debug-03.png)

### TODO
* 部分**SDK**接口不存在 
```hava
// 将会报错
import com.intellij.testFramework.fixtures.LightCodeInsightFixtureTestCase;  
```


### 参考资料
------
* [How debug a custom plugin in PHPStorm](https://intellij-support.jetbrains.com/hc/en-us/community/posts/206232149-How-debug-a-custom-plugin-in-PHPStorm)

* [AppCode plugin development: How to run/debug from Idea CE](https://intellij-support.jetbrains.com/hc/en-us/community/posts/205812589-AppCode-plugin-development-How-to-run-debug-from-Idea-CE)