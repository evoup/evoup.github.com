---
layout: post
title: "VS2010 LINK:fatal error LNK1123"
date: 2014-02-10 15:22
comments: true
categories: c-language
---

因为要看cocostudio的实例代码，在原有vs2010的基础上又安装了vs2012，出现了项目不能编译的问题OMG
` LINK : fatal error LNK1123: 转换到 COFF 期间失败: 文件无效或损坏 `

<!-- more -->

> VS2010在经历一些更新后，建立Win32 Console Project时会出“error LNK1123” 错误，解决方案为将 项目|项目属性|配置属性|清单工具|输入和输出|嵌入清单 “是”改为“否”即可，但是没新建一个项目都要这样设置一次。
> 在建立VS2010 Win32 Project项目时，按照上面解决方案依然发生了“error LNK1123”错误，经过上网查资料，解决方案为：
> 第一步：与上相同。
> 第二步：将 项目|项目属性|配置属性|连接器|清单文件|嵌入清单 “是”改为“否”。
> 第三步：一般计算机经过上两步设置就能解决问题了，但是如果还有问题，那就按一下方法解决：
> 计算机是否为64bit操作系统，如是，继续2。
> 查找是否有两个cvtres.exe。一个是C:\Program Files (x86)\Microsoft Visual Studio 10.0\VC\bin\cvtres.exe， 另一个是C:\Windows\Microsoft.NET\Framework\v4.0.30319\cvtres.exe。右键属性|详细信息 查看两者版本号，删除/重命名较旧的版本，或者重新设置Path变量。
> 删除或者重命名比较旧的版本的cvtres.exe即可。
