---
layout: post
title: "zookeeper伪分布配置安装"
date: 2014-07-30 17:22
comments: true
categories: zookeeper
---

###场景
我的开发机是在vmware的centos6.5上搭建了一个hadoop节点来做开发，现在想在项目中加入zookeeper做HA的功能（一个leader，多个follower），一开始想到的是再搞一台机器实现完全分布，为了一个zookeeper其实不用再搞一台机器节省资源，后来考虑了下其实还有个伪分布的概念，所谓伪分布就是在一台机器上启动多个实例。下文详细描述如何在一台机器上启动多个zookeeper实例实现伪分布zk集群。

<!-- more -->

###伪分布集群安装配置
准备一台机器，假定IP为192.168.216.198。

####下载安装软件

```bash
$ cd /home/hadoop/software
$ wget http://mirror.bit.edu.cn/apache/zookeeper/zookeeper-3.4.5/zookeeper-3.4.5.tar.gz
$ tar xzf zookeeper-3.4.5.tar.gz
$ sudo mv zookeeper-3.4.5 /usr/local/
$ sudo mv /usr/local/zookeeper-3.4.5/ /usr/local/zookeeper
```

####配置三个实例的myid

```bash
$ mkdir -p /home/hadoop/zoo/zk1 /home/hadoop/zoo/zk2 /home/hadoop/zoo/zk3
$ echo "1" > /home/hadoop/zoo/zk1/myid
$ echo "2" > /home/hadoop/zoo/zk2/myid
$ echo "3" > /home/hadoop/zoo/zk3/myid
```

####分配制作三个配置文件

```bash
$ sudo cp /usr/local/zookeeper/conf/zoo_sample.cfg /usr/local/zookeeper/conf/zoo1.cfg
$ sudo cp /usr/local/zookeeper/conf/zoo_sample.cfg /usr/local/zookeeper/conf/zoo2.cfg
$ sudo cp /usr/local/zookeeper/conf/zoo_sample.cfg /usr/local/zookeeper/conf/zoo3.cfg

$ sudo vim /usr/local/zookeeper/conf/zoo1.cfg 
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/home/hadoop/zoo/zk1/
clientPort=2181
server.1=192.168.216.198:2889:3888
server.2=192.168.216.198:2889:3889
server.3=192.168.216.198:2890:3890

$sudo vim usr/local/zookeeper/conf/zoo2.cfg 
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/home/hadoop/zoo/zk2/
clientPort=2182
server.1=192.168.216.198:2889:3888
server.2=192.168.216.198:2889:3889
server.3=192.168.216.198:2890:3890

$sudo vim /usr/local/zookeeper/conf/zoo3.cfg
/usr/local/zookeeper/conf/zoo3.cfg:tickTime=2000
/usr/local/zookeeper/conf/zoo3.cfg:initLimit=10
/usr/local/zookeeper/conf/zoo3.cfg:syncLimit=5
/usr/local/zookeeper/conf/zoo3.cfg:dataDir=/home/hadoop/zoo/zk3/
/usr/local/zookeeper/conf/zoo3.cfg:clientPort=2183
/usr/local/zookeeper/conf/zoo3.cfg:server.1=192.168.216.198:2889:3888
/usr/local/zookeeper/conf/zoo3.cfg:server.2=192.168.216.198:2889:3889
/usr/local/zookeeper/conf/zoo3.cfg:server.3=192.168.216.198:2890:3890
```

####给当前用户账户访问权限（可选）

```bash
$ sudo chown -R hadoop:hadoop /usr/local/zookeeper
```

####启动集群

```bash
$ /usr/local/zookeeper/bin/zkServer.sh start /usr/local/zookeeper/conf/zoo1.cfg
JMX enabled by default
Using config: /usr/local/zookeeper/conf/zoo1.cfg
Starting zookeeper ... STARTED
$ /usr/local/zookeeper/bin/zkServer.sh start /usr/local/zookeeper/conf/zoo2.cfg
JMX enabled by default
Using config: /usr/local/zookeeper/conf/zoo2.cfg
Starting zookeeper ... STARTED
$ /usr/local/zookeeper/bin/zkServer.sh start /usr/local/zookeeper/conf/zoo3.cfg
JMX enabled by default
Using config: /usr/local/zookeeper/conf/zoo3.cfg
Starting zookeeper ... STARTED
[hadoop@mdn4namenode1 zk1]$ jps
3115 Jps
3084 QuorumPeerMain
3043 QuorumPeerMain
3015 QuorumPeerMain
```

####查看zookeeper工作文件目录结构

```bash
$ ls -R /home/hadoop/zoo/
/home/hadoop/zoo/:
zk1  zk2  zk3

/home/hadoop/zoo/zk1:
myid  version-2  zookeeper.out  zookeeper_server.pid

/home/hadoop/zoo/zk1/version-2:
acceptedEpoch  currentEpoch  snapshot.0

/home/hadoop/zoo/zk2:
myid  version-2  zookeeper_server.pid

/home/hadoop/zoo/zk2/version-2:
acceptedEpoch  currentEpoch  snapshot.0

/home/hadoop/zoo/zk3:
myid  version-2  zookeeper_server.pid

/home/hadoop/zoo/zk3/version-2:
acceptedEpoch  currentEpoch  snapshot.100000000
```

####查看zookeeper运行情况

```bash
/usr/local/zookeeper/bin/zkServer.sh status /usr/local/zookeeper/conf/zoo1.cfg
/usr/local/zookeeper/bin/zkServer.sh status /usr/local/zookeeper/conf/zoo2.cfg
/usr/local/zookeeper/bin/zkServer.sh status /usr/local/zookeeper/conf/zoo3.cfg
```

报错，需要修正zkServer.sh
把

```bash
    #STAT=`$JAVA "-Dzookeeper.log.dir=${ZOO_LOG_DIR}" "-Dzookeeper.root.logger=${ZOO_LOG4J_PROP}" \
             #-cp "$CLASSPATH" $JVMFLAGS org.apache.zookeeper.client.FourLetterWordMain localhost \
             #$(grep "^[[:space:]]*clientPort" "$ZOOCFG" | sed -e 's/.*=//') srvr 2> /dev/null    \
          #| grep Mode`
```

给注释了，然后在其下加一行

```bash
    STAT=`echo stat | nc 127.0.0.1 $(grep clientPort "$ZOOCFG" | sed -e 's/.*=//') 2> /dev/null| grep Mode`
```

这样就妥妥的了:)

```bash
$ /usr/local/zookeeper/bin/zkServer.sh status /usr/local/zookeeper/conf/zoo1.cfg
JMX enabled by default
Using config: /usr/local/zookeeper/conf/zoo1.cfg
Mode: follower
$ /usr/local/zookeeper/bin/zkServer.sh status /usr/local/zookeeper/conf/zoo2.cfg
JMX enabled by default
Using config: /usr/local/zookeeper/conf/zoo2.cfg
Mode: leader
$ /usr/local/zookeeper/bin/zkServer.sh status /usr/local/zookeeper/conf/zoo3.cfg
JMX enabled by default
Using config: /usr/local/zookeeper/conf/zoo3.cfg
Mode: follower
```

####加入启动项

```bash
sudo vim /etc/rc.d/rc.local
#!/bin/sh
su - hadoop -c "/usr/local/zookeeper/bin/zkServer.sh start /usr/local/zookeeper/conf/zoo1.cfg"
su - hadoop -c "/usr/local/zookeeper/bin/zkServer.sh start /usr/local/zookeeper/conf/zoo2.cfg"
su - hadoop -c "/usr/local/zookeeper/bin/zkServer.sh start /usr/local/zookeeper/conf/zoo3.cfg"
```

收工。

####参考资料
<a href="http://blog.fens.me/hadoop-zookeeper-intro/">《ZooKeeper伪分布式集群安装及使用 | 粉丝日志》</a>


