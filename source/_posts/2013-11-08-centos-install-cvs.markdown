---
layout: post
title: "centos下CVS服务器的配置"
date: 2013-11-08 13:20
comments: true
categories: version_control 
---

由于需要回顾以前在自己的cvs上建设的一些项目,把其中有用的项目直接放到github上方便调用，现在需要重拾cvs这个历史悠久的版本管理工具了。

<!-- more -->

在centos6上建设cvs服务器是比较方便的。大体分为几个步骤，xinetd->cvs->配置帐号->配置xinetd下pcvsserver文件->启动cvs服务端

####1.安装xinetd

```bash
rpm -q xinetd 
```

没有安装的话

```bash 
yum install xinetd
```

####2.安装cvs

```bash
rpm -q cvs
```

我的系统显示
cvs-1.11.23-15.el6.x86_64
如果没有安装的话可以

```bash
yum install cvs
```

####3.配置帐号
切换到root，执行：

```bash
groupadd cvs
useradd -g cvs cvsroot
passwd cvsroot
```

这样帐号就配置好了。

####4.修改cvs的配置文件/etc/xinetd.d/pcvsserver

```bash
# default: off
# description: The CVS service can record the history of your source \
#              files. CVS stores all the versions of a file in a single \
#              file in a clever way that only stores the differences \
#              between versions.
service cvspserver
{
        disable                 = no
        port                    = 2401
        socket_type             = stream
        protocol                = tcp
        wait                    = no
        user                    = root
        passenv                 = PATH
        server                  = /usr/bin/cvs
        env                     = HOME=/home/cvsroot
        server_args             = -f --allow-root=/home/cvsroot pserver
#       bind                    = 127.0.0.1
}
```
其中env为cvs的环境变量地址，server_args指定你项目的源代码的保存路径和认证方式，这里pserver为基于密码的认证方式。


####5.初始化和启动
初始化项目仓库的命令为

```bash
cvs -d /home/cvsroot init
```

就是在刚才配置的env = HOME=/home/cvsroot目录长噢乖创建代码仓库

接下来重启xinetd

```bash
/etc/rc.d/init.d/xinetd restart
```

测试cvs是否存在
lsof -p 2401
或者
netstat -l | grep csv
不能使用ps aux | grep cvs，因为这是在xinetd下！

安装就完成了。

接下来还要提一下客户端tortoise cvs使用的问题，需要把cvs.exe的路径加到环境变量PATH中方可使用或者把cvsnt这款软件的cvs.exe拷贝到tortoise cvs安装目录中，否则会报句柄错误。


####参考链接：

http://blog.csdn.net/lidongtang/article/details/8165574

