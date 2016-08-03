---
layout: post
title: "freebsd下hbase安装"
date: 2011-07-30 21:51
comments: true
categories:           hbase
---
安装JDK补遗,port中该文件已经无法获取。关键是下载到diablo-caffe-freebsd7-i386-1.6.0_07-b02.tar.bz

http://www.freebsdfoundation.org/cgi-bin/download?download=diablo-caffe-freebsd7-amd64-1.6.0_07-b02.tar.bz2

<!-- more -->

下完就放到/usr/port/distfile

进到/usr/ports/java/diablo-jdk16之后也要把timezone那个选取消。

再装jdk16/usr/port/java/jdk16

```bash
sudo axel http://www.java.net/download/jdk6/6u3/promoted/b05/jdk-6u3-fcs-src-b05-jrl-24_sep_2007.jar 
sudo mv jdk-6u3-fcs-src-b05-jrl-24_sep_2007.jar php-5.3.8.tar.bz2 /usr/ports/distfiles/
```

装apache-ant，自动的，如果不行cd /usr/port/devel/apache-ant/ sudo make install clean

而且要有足够的SWAP空间！

```bash
setenv JAVA_HOME /usr/local/jdk1.6.0/
```
PATH我没有设置

然后是hbase的安装，拿到权威手册翻一翻就能知道

单个节点的启动方法

```bash
sudo ./bin/start-hbase.sh
sudo ./bin/hbase-daemon.sh start thrift
```
hshell的进入

```bash
sudo ./bin/hbase shell
```

 
