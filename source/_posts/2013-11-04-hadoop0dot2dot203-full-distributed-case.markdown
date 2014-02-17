---
layout: post
title: "HADOOP0.20.203完全分布式搭建(VMware版)"
date: 2013-11-04 15:28
comments: true
categories:          hadoop
---

vmware版本8.0.4 build-744019
首先准备3台虚拟机

![Alt text](/images/evoup/hadoop_vmware.png)

```
 ,''''''''''''':'''''''''''''''''':'''''''''''''''''''''''''''''''''''|
 |    usage    |        IP        |             Hostname              |
 |             |                  |                                   |
 |'''''''''''''|''''''''''''''''''|'''''''''''''''''''''''''''''''''''|
 | namenode    | 192.168.174.132  |           mdn2.net                |
 |             |                  |                                   |
 |'''''''''''''|''''''''''''''''''|'''''''''''''''''''''''''''''''''''|
 | datanode01  | 192.168.174.135  |        mdn2_datanode1.net         |
 |             |                  |                                   |
 |'''''''''''''|''''''''''''''''''|'''''''''''''''''''''''''''''''''''|
 | datanode02  | 192.168.174.136  |        mdn2_datanode2.net         |
 |             |                  |                                   |
  '''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
```

 <!-- more -->

Ps:
为节省资源关闭点不必要的服务
```bash
#!/bin/bash
for SERVICES in abrtd acpid auditd avahi-daemon cpuspeed haldaemon mdmonitor messagebus udev-post;
do /sbin/chkconfig ${SERVICES} off;
done
```

###准备工作

先安装JDK1.6
linux:先把已经安装的openjdk卸载,安装sun jdk1.6 （j2se就够了）
```bash
#rpm -qa | grep jdk
java-1.6.0-openjdk-1.6.0.0-1.28.1.10.9.el5_8
#rpm -e java-1.6.0-openjdk-1.6.0.0-1.28.1.10.9.el5_8
#sudo chmod +x jdk-6u35-linux-x64-rpm.bin
#./jdk-6u35-linux-x64-rpm.bin
```

freebsd:直接在/usr/port/java/diablo-jdk16，不要装jdk16，打好几个补丁还编译通不过，浪费时间！）

hadoop所有操作都是用hadoop帐号，下面添加

```bash
linux:# groupadd hadoop
freebsd:# pw groupadd hadoop
linux:# useradd -r -g hadoop -d /home/hadoop -m -s /bin/bash hadoop
freebsd:# pw adduser hadoop -g hadoop -d /home/hadoop -m -s /bin/bash

all:# mkdir -p /u01/app
all:# chgrp -R hadoop /u01/app
all:# chown -R hadoop /u01/app
```

环境变量
```bash
all:$ vi ~/.profile
all:export HADOOP_HOME=/u01/app/hadoop
all:export HBASE_HOME=/u01/app/hbase
```


进行免密码的ssh登录设置
```bash
ssh-keygen -t dsa -P '' -f ~/.ssh/id_dsa
cat ~/.ssh/id_dsa.pub >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```
到此准备工作完成

安装Hadoop
```bash
all:$ cd /u01/app
all:$ tar zxf hadoop-0.20.203.0rc1.tar.gz
all:$ ln -s hadoop-0.20.203.0 hadoop
```

###正式开始

编辑所有机器的/etc/hosts文件（host的centos下为/etc/sysconfig/network，bsd的要设置/etc/rc.conf）
(PS:也可以选择现在namenode上编辑好了，分发到其他机器上去)

```bash
# Do not remove the following line, or various programs
# that require network functionality will fail.

127.0.0.1       localhost.localdomain localhost
::1             localhost6.localdomain6 localhost6
192.168.174.132 mdn2.net
192.168.174.135 mdn2_datanode1.net
192.168.174.136 mdn2_datanode2.net
```

假定已经安装好了JAVA，编辑hadoop帐号的profile文件加入如下代码
```bash
export HADOOP_HOME=/u01/app/hadoop
export HBASE_HOME=/u01/app/hbase
export PATH="/usr/java/jdk1.6.0_37/bin/:$PATH"
export JAVA_HOME="/usr/java/jdk1.6.0_37/bin/"
```

下载hadoop解压之后，在hadoop-env.sh指定java的目录
```bash
export JAVA_HOME=/usr/java/jdk1.6.0_37/
```

再编辑core-site.xml
```xml
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>

<!-- Put site-specific property overrides in this file. -->

<configuration>
    <property>
        <name>fs.default.name</name>
        <value>hdfs://mdn2.net:9024</value>
    </property>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/u01/app/hadoopTmp</value>
    </property>
</configuration>
```

注意：hadoop.tmp.dir是hadoop文件系统依赖的基础配置，很多路径都依赖它。它默认的位置是在/tmp/{$user}下面，在local和hdfs都会建有相同的目录，但是在/tmp路径下的存储是不安全的，因为linux一次重启，文件就可能被删除。导致namenode启动不起来。

再编辑hdfs-site.xml
```xml
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>

<!-- Put site-specific property overrides in this file. -->

<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
    <property>
        <name>dfs.data.dir</name>
        <value>/u01/app/hdfsdata</value>
    </property>
</configuration>
```

注意：dfs.data.dir为hdfs实际存放数据的路径，这个配置只对本地有效，中间可以用,连接多个目录

master里
```bash
mdn2.net
```

slaves里
```bash
mdn2_datanode1.net
mdn2_datanode2.net
```

注意点：
修改hadoop-0.20.203.0/bin下的hadoop.
vi  hadoop
查找 –jvm . vi 下的命令模式： :/-jvm
将-jvm server改成 –server .
因为JDK1.6已经废除了一个参数-jvm,如果不修改的话，无法启动数据节点。

到namenode上格式化hdfs
```bash
/bin/hadoop namenode -format
```
注意9024为hdfs通讯端口，完全分布式环境下，可以直接将防火墙关闭
```bash
sudo /etc/init.d/iptables stop
sudo /sbin/chkconfig iptables off
```
启动：
```bash
bin/start-all.sh
```
或者只启动dfs和mapreduce
```bash
bin/start-dfs.sh
bin/start-mapred.sh
```
最后发一个jps的进程图

![Alt text](/images/evoup/hadoop_vmware01.png)





