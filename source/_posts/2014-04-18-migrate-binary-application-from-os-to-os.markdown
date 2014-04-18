---
layout: post
title: "在操作系统之间迁移二进制程序"
date: 2013-04-18 17:52
comments: true
categories: freebsd
---

###写在前面
乍一看又标题党了，但以下文章虽然提到了一个过时的软件pcc2.0，实际我们只是用通过研究程序内部的工作机制，进行举一反三，以再今后的工作中能够起到触类旁通、抛砖引玉的功效。<br>
单位有台机器HostA上安装了鲜为人知的软件叫做pcc2.0，这是一个php的编译器，虽然有许多bug，仍旧采用人工hack的方式使用它。但是目前已经找不到安装包了，怎么办？难道要叫运维帮忙克隆系统吗，有没有办法把程序直接移植到另一台机器HostB上去？答案是肯定的。

<!-- more -->
###程序定位和复制
####主程序的复制
这个程序就是pcc，首先找一下pcc的路径。
```sh
$ whereis pcc
pcc: /usr/bin/pcc /usr/ports/lang/pcc
```

可见pcc所在/usr/bin/目录下，先把这个复制过去
```sh
$ sudo scp user@hostA:/usr/bin/pcc /usr/bin/
```
####程序依赖库的复制
如果这个程序是静态方式编译的，就这样完成了，而如果是动态编译的话，就需要拷贝其他动态库使它能够运行了。怎么确定是否是静态编译还是动态编译？很简单:使用ldd(在HostA上)，我们看一下吧：
```sh
$ ldd /usr/bin/pcc
/usr/bin/pcc:
        libphp-runtime_u.so => /usr/lib/libphp-runtime_u.so (0x28083000)
        libprofiler_u.so => /usr/lib/libprofiler_u.so (0x280ee000)
        libwebconnect_u.so => /usr/lib/libwebconnect_u.so (0x280f2000)
        libphpeval_u.so => /usr/lib/libphpeval_u.so (0x28108000)
        libbigloo_u-2.6f.so => /usr/lib/libbigloo_u-2.6f.so (0x28250000)
        libbigloogc-2.6f.so => /usr/lib/libbigloogc-2.6f.so (0x2833a000)
        libm.so.4 => /lib/libm.so.4 (0x2835c000)
        libc.so.6 => /lib/libc.so.6 (0x28372000)
```
好的，我们登进HostB，可以看到小小一个pcc软件依赖了这么多的so文件，所以这些文件以及这些文件的依赖库都是要复制过去的。值得一提的是，pcc使用了libbigloo库，而它的版本是2.6f，所以我们要先安装好。

```sh
$ cd /home/software
$ fetch https://pkgs.fedoraproject.org/repo/pkgs/bigloo/bigloo2.6f.tar.gz/bc99b1919adee864dd371aeebc36862c/bigloo2.6f.tar.gz
$ tar xzf bigloo2.6f.tar.gz
$ cd bigloo2.6f
$ ./configure --jvm=no
*** ERROR:configure:the C compiler (/usr/local/bin/gcc44) does not seem to work. Aborting
$ whereis gcc
gcc: /usr/bin/gcc /usr/share/man/man1/gcc.1.gz /usr/src/contrib/gcc
$ sudo ln -s /usr/bin/gcc /usr/local/bin/gcc44
$ ./configure --jvm=no
$ gmake
$ sudo gmake install
```

安装完成返回来继续复制，要注意已经存在的不能去盖了
```sh
$ ldd /usr/bin/pcc
/usr/bin/pcc:
        libphp-runtime_u.so => not found (0x0)
        libprofiler_u.so => not found (0x0)
        libwebconnect_u.so => not found (0x0)
        libphpeval_u.so => not found (0x0)
        libbigloo_u-2.6f.so => /usr/local/lib/libbigloo_u-2.6f.so (0x28083000)
        libbigloogc-2.6f.so => /usr/local/lib/libbigloogc-2.6f.so (0x2816d000)
        libm.so.4 => /lib/libm.so.4 (0x2818f000)
        libc.so.6 => /lib/libc.so.6 (0x281a5000)
```

好的，只要not found的我们从HostA复制过来
```sh
sudo scp yin@211.136.104.179:/usr/lib/libphp-runtime_u.so /usr/lib/
sudo scp yin@211.136.104.179:/usr/lib/libprofiler_u.so /usr/lib/
sudo scp yin@211.136.104.179:/usr/lib/libwebconnect_u.so /usr/lib/
sudo scp yin@211.136.104.179:/usr/lib/libphpeval_u.so /usr/lib/
```
然后对于每个新的so，需要再进一步检查依赖，如果没有的也要复制过来

