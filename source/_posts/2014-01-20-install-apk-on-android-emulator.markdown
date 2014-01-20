---
layout: post
title: "在android模拟器上安装apk软件"
date: 2014-01-20 14:56
comments: true
categories: [android]
---
###前提
在android模拟器上安装文件很简单,这个简单要求2个前提
1）已经安装好了ADK（android sdk）
2）配置好virtual device

<!-- more -->

###准备工作
假设您已经安装好了ADK，运行Eclipse选择Window菜单： 

![Alt text](/images/evoup/android_emulator_install_apk_first.png)

选择下方的Android Virtual Device Manager->new,我们创建一个android2.2的通用虚拟设备，屏幕选最小的我这里方便截图（一般会选择3.7WVGA,后期根据自己的需要可以做任意选择），然后取名这个模拟器为testAVD。

![Alt text](/images/evoup/android_emulator_create_new_one.png)

点击OK完成，然后选择testAVD并Start启动模拟器，然后选择launch。第一次启动很慢，至于到底需要多少时间取决于机器的性能。

![Alt text](/images/evoup/android_emulator_starting.png)

这样就算进去了

![Alt text](/images/evoup/android_emulator_entered.png)

###安装apk
进入ADK的platform_tools目录下，运行` adb install apk文件 `就可以完成安装。

以安装微信安卓版本为例，假设我下载到了C:\weixin510android360.apk

![Alt text](/images/evoup/android_emulator_entered.png)

我这里比较慢，可能是因为选择的模拟器内存参数没有调大导致的。

![Alt text](/images/evoup/android_emulator_install_apk.png)

等待安装完成吧

![Alt text](/images/evoup/android_emulator_install_apk_done.png)

![Alt text](/images/evoup/android_emulator_install_apk_done2.png)

