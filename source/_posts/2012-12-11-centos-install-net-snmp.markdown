---
layout: post
title: "centos上安装net-snmp"
date: 2012-12-11 11:08
comments: true
categories: [monitor]
---
###引子
> Net-SNMP是一个免费的、开放源码的SNMP实现，以前称为UCD-SNMP。它包括agent和多个管理工具的源代码，支持多种扩展方式。[1]不仅扩展了获取方式，而且对于数据类型也有一定的扩展。

以上文字摘自百度百科。而所谓snmp agent，个人理解就是能够实现snmp的read和snmp和get等标准snmp协议交互操作的容器，它可以提供默认的MIB和自定义的MIB。

###centos上安装net-snmp的过程

```sh
$ sudo yum install net-snmp
```


###配置snmpd
配置的方法有2种，一个是` snmpconf -g basic_setup `,一个如下的方式（注:前一个我由于时间关系没有试过）：

安装完成后编辑/etc/snmp/snmpd.conf文件，更改如下配置：

主要是控制那台机器可以访问snmp

####1、查找以下字段：

```sh
sec.name source          community
com2sec notConfigUser default       public
```

将"comunity"字段改为你要设置的密码.比如"public".                                            
将“default”改为你想哪台机器可以看到你的snmp信息,如127.0.0.1

####2、查找以下字段：

```sh
# Finally, grant the group read-only access to the systemview view.
#       group          context sec.model sec.level prefix read   write notif
access notConfigGroup ""      any       noauth    exact all none none
```

将"read"字段改为all.
####3、查找以下字段：

```sh
#           incl/excl subtree                          mask
#view all    included .1                               80
```

将该行前面的"#"去掉.
保存关闭.

###snmpwalk和snmpget的区别
snmpwalk是对OID的遍历，必须从根节点一次遍历，而不能通过某个叶子节点遍历。
而snmpget是取得具体的OID的值，必须是叶子节点。

###用snmpwalk进行监控数据的获取
编辑完成之后可以测试，需要用到snmpwalk和snmpget

缺少snmpwalk和snmpget的可以安装

```sh
$ sudo yum install net-snmp-utils
```

启动snmpd

```sh
$ sudo /etc/init.d/snmpd start
$ sudo /sbin/chkconfig snmpd on
```

测试是否开启snmp服务

```sh
$ snmpwalk -v 2c -c public 127.0.0.1 if
IF-MIB::ifIndex.1 = INTEGER: 1
IF-MIB::ifIndex.2 = INTEGER: 2
IF-MIB::ifDescr.1 = STRING: lo
IF-MIB::ifDescr.2 = STRING: eth0
IF-MIB::ifType.1 = INTEGER: softwareLoopback(24)
IF-MIB::ifType.2 = INTEGER: ethernetCsmacd(6)
IF-MIB::ifMtu.1 = INTEGER: 16436
IF-MIB::ifMtu.2 = INTEGER: 1500
IF-MIB::ifSpeed.1 = Gauge32: 10000000
IF-MIB::ifSpeed.2 = Gauge32: 1000000000
IF-MIB::ifPhysAddress.1 = STRING:
IF-MIB::ifPhysAddress.2 = STRING: 0:c:29:e6:ed:27
IF-MIB::ifAdminStatus.1 = INTEGER: up(1)
IF-MIB::ifAdminStatus.2 = INTEGER: up(1)
IF-MIB::ifOperStatus.1 = INTEGER: up(1)
IF-MIB::ifOperStatus.2 = INTEGER: up(1)
IF-MIB::ifLastChange.1 = Timeticks: (0) 0:00:00.00
IF-MIB::ifLastChange.2 = Timeticks: (0) 0:00:00.00
IF-MIB::ifInOctets.1 = Counter32: 784
IF-MIB::ifInOctets.2 = Counter32: 4216979
IF-MIB::ifInUcastPkts.1 = Counter32: 14
IF-MIB::ifInUcastPkts.2 = Counter32: 5593
IF-MIB::ifInNUcastPkts.1 = Counter32: 0
IF-MIB::ifInNUcastPkts.2 = Counter32: 0
IF-MIB::ifInDiscards.1 = Counter32: 0
IF-MIB::ifInDiscards.2 = Counter32: 0
IF-MIB::ifInErrors.1 = Counter32: 0
IF-MIB::ifInErrors.2 = Counter32: 0
IF-MIB::ifInUnknownProtos.1 = Counter32: 0
IF-MIB::ifInUnknownProtos.2 = Counter32: 0
IF-MIB::ifOutOctets.1 = Counter32: 784
IF-MIB::ifOutOctets.2 = Counter32: 417463
IF-MIB::ifOutUcastPkts.1 = Counter32: 14
IF-MIB::ifOutUcastPkts.2 = Counter32: 3968
IF-MIB::ifOutNUcastPkts.1 = Counter32: 0
IF-MIB::ifOutNUcastPkts.2 = Counter32: 0
IF-MIB::ifOutDiscards.1 = Counter32: 0
IF-MIB::ifOutDiscards.2 = Counter32: 0
IF-MIB::ifOutErrors.1 = Counter32: 0
IF-MIB::ifOutErrors.2 = Counter32: 0
IF-MIB::ifOutQLen.1 = Gauge32: 0
IF-MIB::ifOutQLen.2 = Gauge32: 0
IF-MIB::ifSpecific.1 = OID: SNMPv2-SMI::zeroDotZero
IF-MIB::ifSpecific.2 = OID: SNMPv2-SMI::zeroDotZero
```

获取load信息

```sh
$ snmpwalk -v 2c 127.0.0.1 -c public .1.3.6.1.4.1.2021.10.1.3
UCD-SNMP-MIB::laLoad.1 = STRING: 0.03
UCD-SNMP-MIB::laLoad.2 = STRING: 0.03
UCD-SNMP-MIB::laLoad.3 = STRING: 0.00
```

同样可以尝试使用snmp版本1来获取

```sh
$ snmpwalk -v 1 -c public localhost .1.3.6.1.4.1.2021.10.1.3
UCD-SNMP-MIB::laLoad.1 = STRING: 0.01
UCD-SNMP-MIB::laLoad.2 = STRING: 0.02
UCD-SNMP-MIB::laLoad.3 = STRING: 0.00
```


###对外开放SNMP服务
得知本机已经安装完成，但是根据之前的设置，应该只有本机才有snmp的访问权限。可以通过修改/etc/snmp/snmpd.conf把刚才设置的127.0.0.1改为default


确认本机开放snmp端口，由于snmp协议基于udp，所以采用基于tcp的telnet来测试是不奏效的，可以利用nc来测试，如下

```sh
$ nc -uvz 192.168.216.177 161
Connection to 192.168.216.177 161 port [udp/snmp] succeeded!
```

如果打开了IPTABLES,确保本机的udp161端口对外开放，或者说入站通讯端口为udp161

```sh
$ sudo /sbin/iptables -I INPUT -p udp --dport 161 -j ACCEPT
$ sudo /etc/rc.d/init.d/iptables save
$ sudo /etc/init.d/iptables restart
```

或者直接在/etc/sysconfig/iptable中增加一行：

```sh
-A RH-Firewall-1-INPUT -m state –state NEW -m udp -p udp –dport 161 -j ACCEPT
```

###小结
出于快速分析的目的，本文简单阐述了net-snmp在centos下的安装步骤，更加详细的内容可以参见net-snmp的[官方网站](http://www.net-snmp.org/)

