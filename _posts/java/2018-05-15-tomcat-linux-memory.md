---
layout: post
title:  "tomcat memory 暂用过高被系统杀死"
date:   2018-05-15 02:32:22 +0800
categories: java
tags: [java, Jetbrains]
---

> 小记部署tomcat, 内存暂用过高被系统杀死


```shell

> top -u tomcat8
  
  PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND
 9783 tomcat8   20   0 2138556 220668   8436 S  0.3 42.1   0:22.34 java

> free -h
              total        used        free      shared  buff/cache   available
Mem:           512M        478M          0B         29M         33M          0B
Swap:           64M         64M          0B

```

...非常的尴尬, 没钱是这样子的


** 加了一段配置 试了下 **
```shell

> vim /usr/share/tomcat8/bin/catalina.sh 

JAVA_OPTS="-server -Xms40m -Xmx40m -XX:PermSize=12m -XX:MaxPermSize=25m -XX:MaxNewSize=25m"

# 确实降了一点..
> top -u tomcat8
  PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND
10500 tomcat8   20   0 1987884 197544  13984 S  0.0 37.7   0:40.49 java

> free -h
              total        used        free      shared  buff/cache   available
Mem:           512M        449M          0B         29M         62M          0B
Swap:           64M         64M          0B
```
