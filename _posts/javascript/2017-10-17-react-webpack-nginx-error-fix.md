---
layout: post
title:  "react's webpack & nginx 部署问题"
date:   2017-10-17 22:49:23 +0800
categories: javascript
tags: [web-front, javascript]
---

# react 部署到 nginx 后 (⌘ + R) 出现 404页面

------

> 原因：使用了 `React-Router` 的 `browserHistory` 模式

## 解决办法

```shell
# nginx conf 中加入
location / {
    try_files $uri $uri/ /index.html;
}
```

# react 部署到 nginx 资源加载出错

------

> 完成了上一步后发现，还是有问题-_-，资源文件（vendor.js 和 bundle.js）加载到了`index.html`的内容，观察了请求发现发出的请求是**http://your-domain.com/your-router-path/vender.js**，也就是说因为使用了`相对url`的问题，为了能够使用同一份缓存，所以选择将`相对url`都改为`绝对url`

## 解决方法
```javascript

// webpack-config.js

// webpackConfig.output.publicPath 配置为对应的域名

//...previous config
  output: {
    publicPath: 'http://your-domain.com'
  }
//...following config
```
# 补充

-------

抄的demo太坑爹了，一堆遗留问题┑(￣Д ￣)┍

# 引用

------

[html-webpack-plugin](https://github.com/jantimon/html-webpack-plugin)