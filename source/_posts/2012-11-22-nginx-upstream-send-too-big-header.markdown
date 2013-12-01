---
layout: post
title: "解决nginx upstream send too big header"
date: 2012-11-22 15:01
comments: true
categories: nginx 
---
遭遇nginx upstream sent too big header while reading response header from upstream

这个问题会导致输出502头信息。

 
在fastcgi_params中加以下2行
```sh
fastcgi_buffer_size 128k;
fastcgi_buffers 8 128k;
```
 
原因是nginx处理header太大了，还有一个原因就是我写程序的时候发header太多了，只能发一次，调试的时候再去做了。
