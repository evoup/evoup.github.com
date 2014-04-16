---
layout: post
title: "win7下cocos2d-js3.0 alpha生成android项目"
date: 2014-03-25 19:37
comments: true
categories: [cocos2d-js,android] 
---

折腾了比较长的时间，主要是从cocos2d-html5入门开始做demo，然后到jsb的部分，发现走不通，于是又去研究了cocos2d-x3.0 v3.0 rc0如何生成android项目，发现发布是可以的，但是安装到模拟器又是闪退。花了两三天时间（间断）终于完成，放这里共享下。

<!-- more -->

###需要的软件
首先当然是下载好cocos2d-js了，放到c:\cocos2d-jsb中：<br>
![Alt text](/images/evoup/cocos2d-js_android/01.png)

然后请安装好这些工具NDK（r9b）, android SDK（20131030）, ANT（1.9.3），Python（2.7.6）<br>
下图中红色框住的部分为实际使用到的软件

![Alt text](/images/evoup/cocos2d-js_android/02.png)

![Alt text](/images/evoup/cocos2d-js_android/03.png)

python要配置好环境变量，在cmd命令调出python，可以看到版本号:<br>

![Alt text](/images/evoup/cocos2d-js_android/04.png)

###安装

万事俱备，到了见证奇迹的时刻,在这之前，请先安装好命令行，其实就是写入几个环境变量<br>
方法是：开始-运行-cmd，进入控制台，然后定位到C:\cocos2d-jsb\Cocos2d-JS-v3.0-alpha,接下来输入:

```sh
python setup.py
```

然后在对应的NDK_ROOT环境变量中输入c:\android\android-ndk-r9b-windows-x86_64\android-ndk-r9b,<br>
在ANDROID_SDK_ROOT环境变量中输入c:\android\adt-bundle-windows-x86_64-20131030\sdk<br>
在ANT_ROOT中环境变量中输入c:\ant\apache-ant-1.9.3\bin<br>
![Alt text](/images/evoup/cocos2d-js_android/05.png)

这些操作完成后，你会发现win7好像并没有加载出这些环境变量，我的方法是重启（嗯！你没看错，反正我是重启才能加载，晕）。<br>

重启完成之后,我们试一下这些环境变量是否OK，注意windows下批处理程序变量的命名方式为%变量名%<br>

![Alt text](/images/evoup/cocos2d-js_android/06.png)

###产生项目
比如说要在c:/projects下创建一个有native编译支持的，名为projectFirst的项目，需要在控制台输入
```sh
%COCOS_CONSOLE_ROOT%/cocos new projectFirst -l js -d c:/projects

```
等一会儿就创建完毕<br>
![Alt text](/images/evoup/cocos2d-js_android/07.png)

###编译项目
如果要编译成android项目，只要执行如下指令即可：<br>
```sh
> cd c:\projects\projectFirst
> %COCOS_CONSOLE_ROOT%/cocos compile -p android
```
之后要耐心等待项目编译完成，顺便说一句，没有vc2012是不能够编译cocos2d-js3.0的，因为现在引擎支持C++11了。<br>
![Alt text](/images/evoup/cocos2d-js_android/08.png)

等待编译完成<br>
![Alt text](/images/evoup/cocos2d-js_android/09.png)

这样就在runtime\android\下生成了projectFirst-debug-unaligned.apk

###在android模拟器测试
这个只要注意如果是2d-js的项目，在模拟器里面是需要打开use host gpu<br>
![Alt text](/images/evoup/cocos2d-js_android/10.png)

启动模拟器后，用adb安装软件进去<br>
```sh
%ANDROID_SDK_ROOT%/platform-tools/adb install c:\projects\projectFirst\runtime\android\projectFirst-debug-unaligned.apk
```
安装完成后运行程序<br>
![Alt text](/images/evoup/cocos2d-js_android/10.png)

告一段落。

