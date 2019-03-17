---
layout: post
title: "设计2D游戏玩家角色"
date: 2014-03-14 23:47
comments: true
categories: [gamedev-cg] 
---
目的，为cocos2d-x的编辑器制作好一个角色，为以后的demo做准备。<br>
最终稿（完成时间2小时+修正1小时，初次尝试，PS）

<!-- more -->

![Alt text](/images/evoup/2dcharactor_cg/01.png)

1.用2b铅笔绘制头部轮廓线，然后用柔边圆压力大小上次，这里的技巧是可以先用魔棒工具选择头部轮廓线图层内部，然后转到头部图层用alt + del填充，接着使用锁定透明色的方法继续用柔边圆压力大小上色，这样叠上去的颜色就不会画到轮廓线外面了。

![Alt text](/images/evoup/2dcharactor_cg/02.png)


2.继续在头部曾上画一层头发轮廓线，然后在后面加一层，按照上面的填充方法如法炮制画出头发

![Alt text](/images/evoup/2dcharactor_cg/03.png)

3.继续画出发箍

![Alt text](/images/evoup/2dcharactor_cg/04.png)

4.马尾，层位于头部以下

![Alt text](/images/evoup/2dcharactor_cg/05.png)

5. 围巾

![Alt text](/images/evoup/2dcharactor_cg/06.png)

6.身体

![Alt text](/images/evoup/2dcharactor_cg/07.png)

7.胸衣

![Alt text](/images/evoup/2dcharactor_cg/08.png)

8.右手臂，画在最顶层

![Alt text](/images/evoup/2dcharactor_cg/09.png)

9.右手下臂

![Alt text](/images/evoup/2dcharactor_cg/10.png)

10.右大腿，身体后的一层

![Alt text](/images/evoup/2dcharactor_cg/11.png)

11.右小腿

![Alt text](/images/evoup/2dcharactor_cg/12.png)

12.左大腿，放到右大腿下层

![Alt text](/images/evoup/2dcharactor_cg/13.png)

13.右下腿

![Alt text](/images/evoup/2dcharactor_cg/14.png)

14.兜裆布

![Alt text](/images/evoup/2dcharactor_cg/15.png)

15.左手整手臂

![Alt text](/images/evoup/2dcharactor_cg/16.png)

16.到最后还不是最后(哈哈，孙艺真的《共犯》台词嘛)，其实下面才是重要的活啊，线稿太粗了，我们要用钢笔来修正。方法是用钢笔工具去照描，可以用ctrl点一下旁边来起新的路径，全部描绘好后，用以下设置

![Alt text](/images/evoup/2dcharactor_cg/18.png)

然后点击右键->描边路径，可以选压力（我这里没有选），见下：

![Alt text](/images/evoup/2dcharactor_cg/17.png)

就可以得到下面的结果了

![Alt text](/images/evoup/2dcharactor_cg/01.png)

如果觉得路径碍事，可以用ctrl+shift+h来关闭显示，或者点击视图->显示->目标路径。

总结不足：下脚轮廓线其实可以去掉，像素外框线可以在圆润点，我只处理了头发那个图层，可以用第16步骤来修正。

<a href="http://pan.baidu.com/s/1hqKFQ1e" target=_BLANK>PS源文件</a>
