---
layout: post
title: "unix下找回svn密码的小方法"
date: 2013-04-10 16:44
comments: true
categories: version_control
---

之前有个项目的svn通过svn manager设置了权限访问，采用了不同的密码，在windows端小海龟的密码是看不到的，但庆幸主要更新和提交svn的任务是在freebsd下进行的，经过一番折腾，找回了密码,方法见下：
<!-- more -->
其实很简单，进入用户的home目录下的.subversion下的auth子目录下，再进入svn.simple，之后你懂的
![Alt text](/images/evoup/svn_pass.png)

居然全是明文哦，这就找回来了：）

