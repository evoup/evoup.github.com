---
layout: post
title: "Win7 Android开发环境搭建之一(SDK+ADT)"
date: 2014-01-17 17:48
comments: true
categories: [android]
---
最近有个HTML5的项目需要部署在android里，先研究下基本功，安装时还是遇到一些问题，如有的文章过时了，自己连搜索测试带整理写了一片文章备查。

<!-- more -->

（2014-09-24 嫌adt配置麻烦的可以直接到android开发者官网下载android-bundle <a href="http://developer.android.com/sdk/index.html#download">点击下载</a>）


###一、安装JDK
安装JDK，我这里的windows7平台，先下载jdk7，然后安装，我的安装路径为C:\Program Files\Java\jdk1.7.0_25\。
安装完成后设置环境变量
开始->搜索程序和文件输入环境变量->编辑系统环境变量->添加以下环境变量：

```sh
JAVA_HOME值为：C:\Program Files\Java\jdk1.7.0_25\
CLASSPATH值为：.;%JAVA_HOME%\lib\tools.jar;%JAVA_HOME%\lib\dt.jar;%JAVA_HOME%\bin;
PATH值为：追加%JAVA_HOME%\bin;
```

###二、安装Eclipse
可以直接去www.eclipse.org下载，64位Windows平台下的下载地址：

http://mirror.bit.edu.cn/eclipse/technology/epp/downloads/release/kepler/SR1/eclipse-cpp-kepler-SR1-win32-x86_64.zip

下载完成解压即可使用。见下图：
![Alt text](/images/evoup/eclipse_dir.png)

###三、安装SDK
首先去下载SDK,这个是等等用来和eclipse的adt插件关联的。

  下载地址如下：
  http://dl.google.com/android/adt/adt-bundle-windows-x86_64-20131030.zip

下载完成后我把它解压到C:\android\adt-bundle-windows-x86_64-20131030,运行adt-bundle-windows-x86_64-20131030下的SDK Manager.exe，勾选几个必要的API版本，Android4.2.2(API17)还有Android2.2(API8)，然后点击Install 11 package开始安装

![Alt text](/images/evoup/adt_wait.png)

如果速度慢，可以尝试用goagent代理进行加速,这里就提一下而以。

![Alt text](/images/evoup/adt_use_proxy.png)

等待下载完成

![Alt text](/images/evoup/adt_done.png)

###SDK配置
接着把tools的路径C:\android\adt-bundle-windows-x86_64-20131030\sdk\tools追加到环境变量PATH中去。
要测试安装sdk是否成功，只需要运行cmd，然后输入

```sh
android -h
```

观察是否有输出正常信息，见下图：
![Alt text](/images/evoup/adt_installed.png)

有其他教程说要设置SDK Location变量，我配置时发现其实不需要，估计是老版本了。

###Eclipse的ADT Package安装
打开Eclipse，指定好工程的位置

![Alt text](/images/evoup/adt_project_dir.png)


选择 Help < Install New Software 菜单。点击右上角的 Add 按钮，弹出 Add Repository 对话框，在 Name 一栏填写：“ADT Plugin”，在 Location 一栏填写：https://dl-ssl.google.com/android/eclipse/

点击 OK。如果不能获取插件信息，就把 https 换成 http 试试。

然后整个勾选Developer Tools（最小安装的话至少要选择其下Android DDMS和Android Development Tools），点击Next，一路接受条款，等待安装插件完毕,中途可能出现安装警告，确认即可。

![Alt text](/images/evoup/eclipse_adt_warning.png)

之后重启Eclipse。


###Eclipse的ADT插件的配置
安装完ADT插件并重启好Eclipse后，点击 Window < Preferences 菜单，点击左侧菜单中的 Android，再点击右侧 Browse...，选择第3步中Android SDK的安装目录，设置完成后，点击 OK。


![Alt text](/images/evoup/eclipse_adt_config.png)

这样 Android 开发环境就搭建好了。

