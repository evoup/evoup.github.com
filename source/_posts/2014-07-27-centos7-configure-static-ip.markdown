---
layout: post
title: "centos7配置静态IP"
date: 2014-07-27 23:02
comments: true
categories: [linux,vmware,centos]
---
centos7按照初始安装时候的developer类型一路装好,在vmware里已经设置为bridge模式，按理说是会自动按照DHCP联网成功的，结果却发现连网卡都没有激活，这里记录下。

<!-- more -->

配置网卡

```bash
vim /etc/sysconfig/network-scripts/ifcfg-ens33
```

原先的配置是这样的

```bash
HWADDR=00:0C:29:2C:E5:30
TYPE=Ethernet
BOOTPROTO=static
DEFROUTE=yes
PEERDNS=yes
PEERROUTES=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_PEERDNS=yes
IPV6_PEERROUTES=yes
IPV6_FAILURE_FATAL=no
NAME=ens33
UUID=ea35e10e-ff50-4a7a-92d5-ab54c58d8b44
ONBOOT=no
```

没想到在centos7里默认是不启动网卡的，奇怪的设定啊，所以网卡才没有激活
改为yes，然后再加上以下几个参数的设置

```bash
IPADDR=192.168.2.181
GATEWAY=192.168.2.1
DNS1=114.114.114.114
DNS2=8.8.8.8
```

其中IPADDR为网卡地址，GATEWAY为网关地址，我在vmware里采用的bridge，所以就是实际的路由器的地址。
DNS1和DNS2这2个至少要填DNS1。
编辑/etc/resolv.conf
并且在下面加入了

```bash
nameserver 114.114.114.114
nameserver 8.8.8.8
```

如果按照上面把/etc/resolv.conf中的dns也配置好之后

```bash
sudo service network restart
```

这样应该就完成配置可以上网了。

ps:也可以使用system-config-network-tui这个软件以GUI方式进行配置，用yum安装即可。
