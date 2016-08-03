---
layout: post
title: "freebsd下安装vmware-tool"
date: 2013-11-05 13:22
comments: true
categories:             freebsd
---

公司自建的vmware虚拟机，每当中餐回来后，由于系统处理待机状态莫名其妙地会比当前时间慢一段时间，导致server端保存数据的时间戳滞后。怎么解决呢？

<!-- more -->

###简单粗暴的方法
首先可以尝试加上ntpdate自动来同步，不过这个方法比较死。

在crontab内加入

```bash
*/15  *   *   *   *   *   root   ntpdate 210.72.145.44
```

以及在/etc/rc.conf中

```bash
ntpdate_enable="YES"
```

###更好的方法
通过安装vmware-tool来一劳永逸的解决。那么我们如何在freebsd中安装vmware-tool呢？
首先需要点击vmware的菜单VM -> Install VMware Tools
vmware-tool是一个perl脚本，先要安装好perl。然后要准备好compat6x-amd64安装包才能继续。

```bash
cd /usr/port/dist
sudo fetch ftp://ftp5.tw.freebsd.org/BSD/FreeBSD/ports/amd64/packages-8-current/Latest/compat6x-amd64.tbz
sudo pkg_add compat6x-amd64.tbz
*******************************************************************************
*                                                                             *
* Do not forget to add COMPAT_FREEBSD6 into                                   *
* your kernel configuration (enabled by default).                             *
*                                                                             *
* To configure and recompile your kernel see:                                 *
* http://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/kernelconfig.html *
*                                                                             *
*******************************************************************************
```

安装依赖库后，切换到root后，执行如下指令：

```bash
mount /cdrom
cd /tmp
tar xzpf /cdrom/vmware-freebsd-tools.tar.gz
cd vmware-tools-distrib
./vmware-install.pl
```

这样vmware-tool就搞定了。
最后为了保险起先，关闭系统后，编辑.vmx文件
加上下面的时间同步的代码

```bash
tools.syncTime = "TRUE"
time.synchronize.continue = "TRUE"
time.synchronize.restore = "TRUE"
time.synchronize.resume.disk = "TRUE"
time.synchronize.shrink = "TRUE"
time.synchronize.tools.startup = "TRUE"
time.synchronize.tools.enable = "TRUE"
time.synchronize.resume.host = "TRUE"
```

参考链接

http://kb.vmware.com/selfservice/microsites/search.do?language=en_US&cmd=displayKC&externalId=1189
http://www.neko6.tk/archives/1067
