---
layout: post
title: "Win7 Android开发环境搭建之二(NDK+CDT)"
date: 2014-03-18 15:08
comments: true
categories: [android,cocos2d-js]
---

由于cocos2d-js用到了android的一些生产软件，比如android sdk、adt、cdt和ndk，前面两个已经在之前的《Win7 Android开发环境搭建之一(SDK+ADT)》说过了，这次主要介绍下怎么安装cdt和ndk。

<!-- more -->

###什么是NDK
> NDK 提供了一系列的工具，帮助开发者快速开发C（或C++）的动态库，并能自动将so 和java 应用一起打包成apk。这些工具对开发者的帮助是巨大的。

###NDK的安装
这个安装其实就是解压（环境变量的添加要等装好了cygwin再添加，这是后话），下载android NDK。http://developer.android.com/tools/sdk/ndk/index.html

我下载的版本是android-ndk-r9d-windows-x86_64.zip<br>
下载后解压到例如：c:\android\android-ndk-r9d-windows-x86_64.zip，结果如下图：<br>
![Alt text](/images/evoup/android_cdt_ndk/01.png)

###安装cygwin与ndk编译
（插一句：cygwin安装后与octopress的环境可能冲突，建议先安装octopress)

####安装cygwin
ndk开发需要gcc环境，在windows下可以用是cygwin模拟linux编译环境。

Cygwin的下载地址：http://www.cygwin.com/

这个直接一路装，选择163的镜像<br>
![Alt text](/images/evoup/android_cdt_ndk/02.png)

选择Devel-Default，然后点击变成Devel-Install，然后持续点击下一步，耐心等待安装完成，时间是比较长的<br>
或者可以选择像网上许多教程说的那样只下载12个包，这个我就不这么做了<br>
![Alt text](/images/evoup/android_cdt_ndk/03.png)

####加入环境变量(可选，这样可以直接在cmd里进行cygwin.bat的操作)
完成之后，需要把cygwin加入环境变量,我这里是安装在了c:/cygwin64,把c:/cygwin64/bin加入环境变量PATH<br>
![Alt text](/images/evoup/android_cdt_ndk/04.png)

####测试cygwin环境
运行c:/cygwin64/Cygwin.bat,输入

```sh
cygcheck -c cygwin
make -v
gcc -v
```

如果显示ok就可以了，之前的报错cygheap base mismatch detected可以先忽略。<br>
![Alt text](/images/evoup/android_cdt_ndk/05.png)

####设置NDK路径：
接下来在windows系统环境变量ndk中添加NDK的路径,刚刚我把NDK解压到C:\android\android-ndk-r9d-windows-x86_64\android-ndk-r9d<br>
然后在环境变量ndk加上它在cygwin shell里的对应路径/cygdrive/c/android/android-ndk-r9d-windows-x86_64/android-ndk-r9d<br>
![Alt text](/images/evoup/android_cdt_ndk/06.png)

编辑完成后，进入cygwin.bat输入<br>
```
cd $ndk
```
可以看到如下结果<br>
![Alt text](/images/evoup/android_cdt_ndk/07.png)

####测试NDK，编译一个例子
例子可以在$ndk/sample/hello-jni中找到<br>
进入cygwin.bat中输入编译指令

```
cd $ndk/sample/hello-jni
$ndk/ndk-build
```

一会儿就能编译完成，见下<br>
![Alt text](/images/evoup/android_cdt_ndk/08.png)

进入libs目录查看结果,观察是否生成了so文件，如果生成则说明你的NDK已经运行正常了。<br>

```
$ cd libs/armeabi/
$ ls
gdb.setup  gdbserver  libhello-jni.so
```

####在Eclipse中完成代码调用

接下来进入Eclipse，测试该项目,创建项目路径为C:\android\project\HelloJni<br>
在Create Android Project时勾选“Create project from existing source”，Root Directory中填C:\android\android-ndk-r9d-windows-x86_64\android-ndk-r9d\samples\hello-jni,如下<br>
![Alt text](/images/evoup/android_cdt_ndk/09.png)<br>
![Alt text](/images/evoup/android_cdt_ndk/10.png)<br>
![Alt text](/images/evoup/android_cdt_ndk/11.png)<br>

看到如下字符串就代表NDK的例子运行成功了<br>
![Alt text](/images/evoup/android_cdt_ndk/12.png)

分析一下调用代码
{% codeblock lang:java HelloJni.java %}
/*
 * Copyright (C) 2009 The Android Open Source Project
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *      http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
package com.example.hellojni;

import android.app.Activity;
import android.widget.TextView;
import android.os.Bundle;


public class HelloJni extends Activity //HelloJni继承自活动对象事件调度类
{
    /** Called when the activity is first created. */
    @Override
    public void onCreate(Bundle savedInstanceState) //重写onCreate方法
    {
        super.onCreate(savedInstanceState);

        /* Create a TextView and set its content.
         * the text is retrieved by calling a native
         * function.
         */
        TextView  tv = new TextView(this); //创建一个文字view
        tv.setText( stringFromJNI() ); //调用自定义方法stringFromJNI
        setContentView(tv);
    }

    /* A native method that is implemented by the
     * 'hello-jni' native library, which is packaged
     * with this application.
     */
    public native String  stringFromJNI(); //有native代表是c++的代码，这里仅仅是声明

    /* This is another native method declaration that is *not*
     * implemented by 'hello-jni'. This is simply to show that
     * you can declare as many native methods in your Java code
     * as you want, their implementation is searched in the
     * currently loaded native libraries only the first time
     * you call them.
     *
     * Trying to call this function will result in a
     * java.lang.UnsatisfiedLinkError exception !
     */
    public native String  unimplementedStringFromJNI(); //同样是C++的代码

    /* this is used to load the 'hello-jni' library on application
     * startup. The library has already been unpacked into
     * /data/data/com.example.hellojni/lib/libhello-jni.so at
     * installation time by the package manager.
     */
    static {
        System.loadLibrary("hello-jni"); //调用刚才ndk生成的hello-jni.so
    }
}
{% endcodeblock %}
C++的代码暂时就不看了，一步一步来，我们还没装cdt，不急着搞清楚，有兴趣的到文章最后去看链接

###什么是CDT
> Eclipse CDT是 Eclipse 插件，它将把 Eclipse 转换为功能强大的 C/C++ IDE。它被设计为将 Java 开发人员喜爱的许多 Eclipse 优秀功能提供给 C/C++ 开发人员，例如项目管理、集成调试、类向导、自动构建、语法着色和代码完成。当 Eclipse 被用作 Java IDE 时，它将利用 JDK 并与之集成。同样地，CDT 将利用标准的 C/C++ 工具并与之集成，例如 g++、make 和 GDB。这使得 CDT 在 Linux 中变得非常流行，这些工具都可在 Linux 中使用并用于大多数 C++ 开发。可以在 Windows 上设置 CDT 以使用相同的工具。目前还在努力将 CDT 与 Microsoft 的 C++ 工具结合使用，以使 CDT 对 Windows C++ 开发人员更有吸引力。总之有了cdt，可以在一个工程里，同时开发C/C++的native代码的java外壳，并且2种代码能够同时编译。

###在Eclipse编辑器中集成CDT

首先在此下载http://www.eclipse.org/cdt/downloads.php<br>
直接下载cdt-master-x.x.x.zip的就可以了，我的是Eclipse版本为Kepler，可以下载以下版本<br>
http://www.eclipse.org/downloads/download.php?file=/tools/cdt/releases/kepler/sr2/cdt-master-8.3.0.zip

![Alt text](/images/evoup/android_cdt_ndk/13.png)

然后通过Eclipse -> Help -> Install New Software -> add -> Achive,选择下载的zip文件

![Alt text](/images/evoup/android_cdt_ndk/14.png)

等待安装完成<br>
![Alt text](/images/evoup/android_cdt_ndk/15.png)

安装完成后如果在Eclipse中可以创建C++项目证明安装完成<br>
![Alt text](/images/evoup/android_cdt_ndk/16.png)

###安装Sequoyah插件（事实证明，kepler+ndk9已经无须这个插件了）
~~ 官网地址 ~~ <br>
~~ https://projects.eclipse.org/projects/tools.sequoyah ~~

~~ 这个地址其实已经作废了,正确的连接如下 ~~ <br>
~~ http://www.mirrorservice.org/sites/download.eclipse.org/eclipseMirror/sequoyah/updates/2.0/ ~~ <br>
~~ Eclipse-Help-Install New Software-Add,在Location里输入上面的地址，name就指定为sequoyah~~ <br>
![Alt text](/images/evoup/android_cdt_ndk/17.png)
![Alt text](/images/evoup/android_cdt_ndk/18.png)

~~ 然后取消下面的Group By Item，之后选择Select All，按下Next安装（没有研究下去，发现这个要求的eclipse版本为3.7以下，我的kepler为4.3，不得以暂时放弃） ~~

###JNI编译环境配置
还是打开HelloJni这个项目，现在转换为C/C++的native的代码<br>
"Eclipse->File->New->Other",选择"C/C++"下的"Convert to a c/C++ Project(Add C/C++ Nature)",然后点击"Next"<br>
![Alt text](/images/evoup/android_cdt_ndk/19.png)

然后在Makefile project中选择Cygwin GCC,点击Finish<br>
![Alt text](/images/evoup/android_cdt_ndk/20.png)

右键项目的Properties，在"C/C++ Build"中取消默认的" Use default build command "的打勾，在Build Command中输入对应bash加空格加ndk-build的路径，我这里是C:\android\android-ndk-r9d-windows-x86_64\android-ndk-r9d\ndk-build<br>
```sh
bash C:\android\android-ndk-r9d-windows-x86_64\android-ndk-r9d\ndk-build
```
![Alt text](/images/evoup/android_cdt_ndk/21.png)

在"C/C++ General"的Paths and Symbols中：在Includes下add新的GNU C依赖路径,根据自己的avd（andriod virtual device）的版本设置对应的头文件，我这里是android17的，则C:\android\android-ndk-r9d-windows-x86_64\android-ndk-r9d\platforms\android-17\arch-arm\usr\include,其他项目可以安装实际环境酌情调整依赖头文件的版本<br>
![Alt text](/images/evoup/android_cdt_ndk/22.png)

验证：
将“/HelloJni/jni/hello-jni.c”中的字符串“Hello from JNI !”如改为“Hello JNI from test!”，运行后在模拟器上输出的字符串改变即说明配置成功。<br> 
![Alt text](/images/evoup/android_cdt_ndk/23.png)

--------------------------

###参考文章
[《Android NDK 安装与配置》](http://my.oschina.net/owen123456/blog/85658)

[《Android: NDK编程入门笔记》](http://www.cnblogs.com/hibraincol/archive/2011/05/30/2063847.html)
