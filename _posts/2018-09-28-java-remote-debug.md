---
layout: post
title: Java远程调试 
date: 2018-9-28
categories: blog
tags: [java,debug]
description: 
---
```javascript
-Xdebug -Xrunjdwp:transport=dt_socket,address=1044,server=y,suspend=n
```
程序启动的时候给jvm加上上面的参数。

然后就可以用Eclipse进行远程调试了。在调试的过程中对代码的修改会反映到程序的运行当中。确实方便了不少。
