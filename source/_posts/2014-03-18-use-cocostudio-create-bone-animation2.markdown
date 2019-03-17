---
layout: post
title: "使用CocoStudio创建骨骼动画（二）"
date: 2014-03-18 22:17
comments: true
categories: [gamedev-cg]
---

上次的《使用CocoStudio创建骨骼动画》讨论到了如何创建骨骼和绑定骨骼关系，这节开始谈怎么创建帧动画。

我把上次的每个图层命名成每个骨骼后，更新后的：<a href="http://pan.baidu.com/s/1mg0UXJi" target=_blank>工作cocostudio源文件下载</a>

<!-- more -->

![Alt text](/images/evoup/cocostudio_bone_animation2/01.png)

点击形体模式之后切换到动画模式<br>
![Alt text](/images/evoup/cocostudio_bone_animation2/02.png)


开始第一个帧动画的制作，右键点击动作列表Animation1<br>
![Alt text](/images/evoup/cocostudio_bone_animation2/03.png)

改名为stand,注意不要改成中文，否则以后写代码调用的时候可能会出现问题<br>
![Alt text](/images/evoup/cocostudio_bone_animation2/04.png)

用鼠标框选全部的帧<br>
![Alt text](/images/evoup/cocostudio_bone_animation2/05.png)

选择复制帧<br>
![Alt text](/images/evoup/cocostudio_bone_animation2/06.png)

在60帧处右键，然后点击粘帖帧<br>
![Alt text](/images/evoup/cocostudio_bone_animation2/07.png)

同样的方法，在30帧处右键点击粘帖帧<br>
![Alt text](/images/evoup/cocostudio_bone_animation2/08.png)

然后把时间轴移到到30帧<br>
![Alt text](/images/evoup/cocostudio_bone_animation2/09.png)

接下来编辑造型了，轻微旋转各个关节<br>
![Alt text](/images/evoup/cocostudio_bone_animation2/10.png)

点击隐藏骨骼，可以展现整体角色<br>
![Alt text](/images/evoup/cocostudio_bone_animation2/11.png)

看效果<br>
![Alt text](/images/evoup/cocostudio_bone_animation2/animation.gif)

然后再创建一个run的动作，如法炮制，未完待续...