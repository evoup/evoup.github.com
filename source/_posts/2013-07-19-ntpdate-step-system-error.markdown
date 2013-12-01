---
layout: post
title: "使用ntpdate同步时间报错: step-systime: Operation not permitted"
date: 2013-7-19 15:35
comments: true
categories: freebsd 
---

今天有台虚拟机进行时间的同步报错
```sh
ntpdate ntp.sjtu.edu.cn

19 Nov 15:38:54 ntpdate[93376]: step-systime: Operation not permitted
```

搜索一把，许多VPS分给用户的虚拟机也存在类似症状，原来是宿主机不允许修改时间，最后通知系统管理员修改搞定。







