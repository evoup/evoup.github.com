---
layout: post
title: "rrdtool的socket通讯接口rrdsrv"
date: 2014-07-10 17:28
comments: true
categories: [monitor,rrdtool]
---

原来以为rrdtool只是个本地数据库，这次写java程序看了几个rrdtool的java实现的源码,在其中一款中发现原来rrdtool居然还支持inetd超级服务器的用法，也就是rrdsrv。以下以freebsd下rrdsrv的配置为例，介绍使用方法：

<!--more-->

假设rrdtool的安装路径在/usr/local/bin/rrdtool，然后存放rrd数据库的路径为/services/rrds/

首先编辑/etc/inetd.conf，加入
```
rrdsrv  stream  tcp nowait  root    /usr/local/bin/rrdtool  rrdtool - /services/rrds/
```

然后再编辑/etc/services，加入
```
rrdsrv 13900/tcp
```
在/etc/rc.conf中
```
inetd_enable="YES"
```

然后
```bash
sudo /etc/rc.d/inetd start
```
这样13900端口就支持使用socket方式的rrdtool命令操作了
```
[yin@yin-arch rrds]>telnet 127.0.0.1 13900
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.

info load.rrd
filename = "load.rrd"
rrd_version = "0003"
step = 15
last_update = 1404984273
header_size = 1000
ds[load].index = 0
ds[load].type = "GAUGE"
ds[load].minimal_heartbeat = 120
ds[load].min = NaN
ds[load].max = NaN
ds[load].last_ds = "5.73242000000000029303e-01"
ds[load].value = 2.0151107237e+00
ds[load].unknown_sec = 0
rra[0].cf = "AVERAGE"
rra[0].rows = 5856
rra[0].cur_row = 5072
rra[0].pdp_per_row = 1
rra[0].xff = 5.0000000000e-01
rra[0].cdp_prep[0].value = NaN
rra[0].cdp_prep[0].unknown_datapoints = 0
rra[1].cf = "AVERAGE"
rra[1].rows = 20160
rra[1].cur_row = 9119
rra[1].pdp_per_row = 4
rra[1].xff = 5.0000000000e-01
rra[1].cdp_prep[0].value = 1.1429347927e+00
rra[1].cdp_prep[0].unknown_datapoints = 0
rra[2].cf = "AVERAGE"
rra[2].rows = 52704
rra[2].cur_row = 46249
rra[2].pdp_per_row = 40
rra[2].xff = 5.0000000000e-01
rra[2].cdp_prep[0].value = 9.7464362627e+00
rra[2].cdp_prep[0].unknown_datapoints = 0
OK u:0.00 s:0.01 r:41.67
```

值得一提的是，除了支持rrdtool的info、create、update等内置命令，更可以调用系统指令cd、mkdir、ls等指令，非常强大。可以看出作者的编程思路非常奇特，居然还可以这样用。于是我借助这个特性，实现了网络rrdtool指令的操作，like memcache：）


