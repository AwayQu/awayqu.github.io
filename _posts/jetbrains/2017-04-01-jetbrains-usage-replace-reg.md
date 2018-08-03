---
layout: post
title:  "jetbrains IDE 常用使用技巧 - 正则捕获组替换"
date:   2017-04-01 22:58:25 +0800
categories: 开发技巧
tags: [jetbrains]
---

# 正则捕获组替换

> 匹配获取部分内容，进行替换

## 示例内容

[点击下载示例内容]({{ site.img_url }}data/jetbrains-regular-exp-replace.json)

```json
[
  {
    "content": "",
    "sid": "62c7e2a9b276d2e03a557aea67f06620",
    "duration": 81,
    "cover": "http://r4.ykimg.com/0541010156455DF36A0A46044FDFE429",
    "title": "\"ä¸ªæ§å­æ¯ééç² cool~\"",
    "source": "youku",
    "priority": 52,
    "pid": 683,
    "datatype": "track",
    "tid": "FKhx5NAHbkW3TDL4/Qn6Lk4K/1o4Pf6frxD5r7e7by8\u003d\n",
    "url": "http://pl-ali.youku.com/playlist/m3u8?ts\u003d1532269138\u0026keyframe\u003d0\u0026m3u8Md5\u003d04820ed2f232f22e3c2b6bf585e2098b\u0026pid\u003d69b81504767483cf\u0026vid\u003dXMTM4NDk5NDUxMg\u003d\u003d\u0026type\u003dhd2\u0026sid\u003d0532269138353211c118a\u0026token\u003d6702\u0026oip\u003d1733178622\u0026ctype\u003d21\u0026ev\u003d1\u0026ep\u003dEHk76lUU/QVzBvVAjToTVWVvfvmAJBxIrBRy63ddRMpUpJey8yTPZisDgI3/pM1f"
  },
  {
    "content": "",
    "sid": "271342692dc110147a7919ea846d67ab",
    "duration": 92,
    "cover": "http://r3.ykimg.com/0541010156455DBC6A0A410453540998",
    "title": "ä¸ªæ§ææç¥çç² æ­å¼ç»¿è²çè¯±æ",
    "source": "youku",
    "priority": 53,
    "pid": 683,
    "datatype": "track",
    "tid": "sxeCrKFk2oDYxrGrYXaFT04K/1o4Pf6frxD5r7e7by8\u003d\n",
    "url": "http://pl-ali.youku.com/playlist/m3u8?ts\u003d1532268991\u0026keyframe\u003d0\u0026m3u8Md5\u003dd4f77cb4b650826d4853ce240f14ecb5\u0026pid\u003d69b81504767483cf\u0026vid\u003dXMTM4NDk5NjMwNA\u003d\u003d\u0026type\u003dhd2\u0026sid\u003d053226899167821dcf855\u0026token\u003d8540\u0026oip\u003d3417915709\u0026ctype\u003d21\u0026ev\u003d1\u0026ep\u003dOZdalbymN3zeBfb95F8PFMrZmJD3EfD95NQrS11f4tf9t+fimneleeRwpWei2wQ8"
  }
]
```
## 目标功能

> 替换 `"url": "http://pl-ali.youku.com/playlist/m3u8?ts\u003d1532269138\u0026keyframe\u003d0\u0026m3u8Md5\u003d04820ed2f232f22e3c2b6bf585e2098b\u0026pid\u003d69b81504767483cf\u0026vid\u003dXMTM4NDk5NDUxMg\u003d\u003d\u0026type\u003dhd2\u0026sid\u003d0532269138353211c118a\u0026token\u003d6702\u0026oip\u003d1733178622\u0026ctype\u003d21\u0026ev\u003d1\u0026ep\u003dEHk76lUU/QVzBvVAjToTVWVvfvmAJBxIrBRy63ddRMpUpJey8yTPZisDgI3/pM1f"` 这串内容， 从中提取`XMTM4NDk5NDUxMg`

# 操作

## 1. 调出 replace 功能

> 快捷键  （ ⌘ +  ⇧ + G ）

![img.png]({{ site.img_url }}imgs/jetbrains/jetbrains-img1.png)

## 2. 书写正则 和 替换内容

**find reg:** `".*vid\\u003d(.*)\\u003d\\u003d\\u0026type.*"`

**replace:** `"url": "$1"`

![img.png]({{ site.img_url }}imgs/jetbrains/jetbrains-img2.png)


## 3.点击replace all

![img.png]({{ site.img_url }}imgs/jetbrains/jetbrains-img3.png)