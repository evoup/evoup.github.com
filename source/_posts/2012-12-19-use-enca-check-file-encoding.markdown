---
layout: post
title: "shell下使用enca指令查看文件编码"
date: 2012-12-19 15:42
comments: true
categories:  shell
---

enca是shell下直接查看文件编码的工具,而转换编码的工具有iconv，这里仅仅介绍enca的安装和使用。

<!-- more -->

####freebsd

```sh
cd /usr/ports/converters/enca
make
sudo make install
```

中途缺少一个record包的，根据包的basename去google搜索下载后放到/usr/port/distfiles然后返回安装即可。
####linux

```sh
yum install -y enca
```

然后按如下方式查看编码格式

使用范例

```sh
$ enca test.txt
Universal transformation format 8 bits; UTF-8
```
