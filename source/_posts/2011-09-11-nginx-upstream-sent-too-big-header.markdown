---
layout: post
title: "遭遇nginx upstream sent too big header"
date: 2011-09-11 23:20
comments: true
categories:         nginx
---

遭遇nginx upstream sent too big header while reading response header from upstream

这个问题会导致输出502头信息。

 
nginx配置中要加以下2行
```shell
fastcgi_buffer_size 128k;
fastcgi_buffers 8 128k;
```

一加果然就不报错了！原因是nginx处理header太大了，还有一个原因就是我写程序的时候发header太多了，只能发一次，程序还要调整啊。