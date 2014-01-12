---
layout: post
title: "win7上搭建cocos2d-x2.2.2开发环境"
date: 2014-01-13 00:25
comments: true
categories: cocos2d-x
---

在win7上搭建cocos2d-x的开发环境，主要用到的开发IDE为visual C++2010，为什么不再使用vc6？第一，cocos2d-x本身不支持。第二，支持C++0x的特性。对于现代游戏游戏引擎来说，这是必要的。安装vc
2010这里就不再讨论。

<!-- more -->

直接讲如何进行cocos2d-x的编译了。
###1）下载
首先当然是要去下载，我这里是cocos2d-x2.2.2版本，下载cocos2d-x2.2.2.zip。这个版本和《cocos2d-x权威指南》有什么关联度的等等研究了就不得而知了。

###2）解压和编译cocos2d-x
接下来是解压,注意不要解压到中文目录，否则接下来生成项目会报错。我解压到了D:\gamedev\cocos2d-x-2.2.2\cocos2d-x-2.2.2。随后进入cocos2d-x2.2.2文件夹下的cocos2d-x2.2.2文件夹。因为我们是使用vc2010，所以就双击cocos2d-win32.vc2010.sln。
进到IDE之后，选择解决方案资源管理器下的cocos2d-win32.vc2010,右击选择配置属性-配置，确保每个项目都为debug，可以看到共有17个项目。然后按下F7开始生成解决方案。
慢慢等，我是等来10来分钟。可以看到17个项目都生成成功了。

###3）创建工程
编译完了就可以创建工程了。当然需要先安装好python，我这里是python2.6.2，有人说要装python2.7，看来并不是必须。然后我们生成工程，运行
```sh
cd D:\gamedev\cocos2d-x-2.2.2\cocos2d-x-2.2.2\tools\project-creator\
create_project.py -project HelloWorld -package com.cocos2d-x.org -language cpp 
```
其中com.cocos2d-x.org是为android项目分配的包名称。运行完毕后，会在D:\gamedev\cocos2d-x-2.2.2\cocos2d-x-2.2.2\project目录下生成ios、android、win32、mac、linux等各种平台的项目。

###4）编译工程
最后进入刚刚取名为HelloWorld的项目中
```sh
cd D:\gamedev\cocos2d-x-2.2.2\cocos2d-x-2.2.2\projects/HelloWorld
```
双击proj.win32下的HelloWorld.sln文件，即可进入工程，没有什么问题的话，就可以按ctrl+F5开始就可以开始进行项目的调试了。



