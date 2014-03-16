---
layout: post
title: "使用cocostudio创建骨骼动画"
date: 2014-03-16 00:28
comments: true
categories: [gamedev-cg]
---

需要给哥么介绍怎么倒腾美术使用的美术创作工具cocostudio（版本1.3）的Animation Editor，所以得先打个头阵研究一番，备忘一下。

<!-- more -->

打开之前我的另一篇《设计2D游戏玩家角色》博客里面提供的PSD源文件，然后开工。明确要怎么分割的部分：<br>

###从PSD文件导出部件
下面给出导出图层的规则，有+的代表有2个图层，复制的时候均采用ctrl+shift+c合并拷贝，再用ctrl+n创建新文件，最后ctrl+c粘帖到新文件上，然后导出为透明png24的方式：

左脚+左脚轮廓线=>左脚<br>
左腿+左腿轮廓线=>左腿<br>
左大腿+左大腿轮廓线=>左大腿<br>
右脚+右脚轮廓线=>右脚<br>
右小腿+右小腿轮廓线=>右小腿<br>
右大腿+右大腿轮廓线=>右大腿<br>
左手整手臂+左手整手臂轮廓线=>左手整手臂<br>
兜裆布=>兜裆布<br>
身体+身体轮廓线=>身体<br>
右手臂+右手臂轮廓线=>右手臂<br>
右手下臂+右手下臂轮廓线=>右手下臂<br>
胸衣=>胸衣<br>
围巾=>围巾<br>
头部+头部轮廓线=头部<br>
眼睛<br>
马尾+马尾轮廓线=>马尾<br>
头发+头发轮廓线=>头发<br>
发箍+发箍轮廓线=》发箍<br>

最后得到这些图素<a href="http://pan.baidu.com/s/1bnzlv9X" target=_BLANK>cuted_player.zip</a>
###打开cocostudio操作

接下来让我们打开cocostudio开始编辑动画，首先进入Animation Editor<br>
![Alt text](/images/evoup/cocostudio_bone_animation/01.png)

然后创建新项目，名为player_swordgirl<br>
![Alt text](/images/evoup/cocostudio_bone_animation/02.png)

在界面右侧的资源窗口右键-“导入”-"导入文件"<br>
![Alt text](/images/evoup/cocostudio_bone_animation/03.png)

完成导入<br>
![Alt text](/images/evoup/cocostudio_bone_animation/04.png)

在主界面上点击创建骨骼，如果再次点击停止创建回到pose mode。我们先创建躯干部份的骨骼。<br>
![Alt text](/images/evoup/cocostudio_bone_animation/05.png)

![Alt text](/images/evoup/cocostudio_bone_animation/06.png)

创建完成之后，把身体的图片赋到该骨骼上，需要注意这里有个bug，就是直接点击骨骼然后把图片拖到右面XX的话，保存项目后，再次打开会发现图片消失了（没想到这个bug1.3还是没修好啊）。
规避这个问题的方法很简单，就是把图片先拖到骨骼旁边，然后右键选择绑定到骨头，这样就不会再消失了。<br>
![Alt text](/images/evoup/cocostudio_bone_animation/07.png)

![Alt text](/images/evoup/cocostudio_bone_animation/08.png)

![Alt text](/images/evoup/cocostudio_bone_animation/09.png)

使用平移工具把身体放到骨骼外围，骨骼嘛自然要待在组织里啊
![Alt text](/images/evoup/cocostudio_bone_animation/10.png)

同样的方法创建出围巾
![Alt text](/images/evoup/cocostudio_bone_animation/11.png)

下面介绍一下骨骼的父关系，这个很好理解的，比如说身体和头，某种程度上说身体就是主躯干，外围设施（比如四肢、头、尾巴等）都是根据身体来依附上去的，这种依附关系的产生，导致移动身体其他外围设施是会跟着发生位移的，那么就可以称这些外围设施和身体有父关系，它的父关系是身体这块骨骼。
右击围巾选择绑定父关系到前一骨骼（身体所在的那块骨骼）<br>
![Alt text](/images/evoup/cocostudio_bone_animation/12.png)

然后试着移动身体，你会发现围巾跟着移动了，很神奇不是吗？：）

接下来如法炮制了，我只讲下哪个绑定哪个<br>
头部绑定父关系到围巾<br>
![Alt text](/images/evoup/cocostudio_bone_animation/13.png)

眼睛绑定父关系到头部，胸衣绑定父关系到头部<br>
![Alt text](/images/evoup/cocostudio_bone_animation/14.png)

头发绑定父关系到头部<br>
![Alt text](/images/evoup/cocostudio_bone_animation/15.png)

发箍绑定父关系到头部<br>
![Alt text](/images/evoup/cocostudio_bone_animation/16.png)

马尾绑定父关系到发箍<br>
![Alt text](/images/evoup/cocostudio_bone_animation/17.png)

右手臂绑定父关系到身体<br>
![Alt text](/images/evoup/cocostudio_bone_animation/18.png)

未完待续

思考：眨眼动画要怎么处理呢？
