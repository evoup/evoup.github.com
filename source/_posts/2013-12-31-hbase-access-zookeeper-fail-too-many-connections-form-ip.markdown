---
layout: post
title: "hbase无法连接zookeeper"
date: 2013-12-31 11:48
comments: true
categories:  [hbase,hadoop,zookeeper]
---

####问题
今天修理hbase问题的时候发现，监控的60010端口的master.jsp就是无法显示，进入log查看发现zookeeper连上了之后马上就断开。
通过telnet测试发现确实有如此情况发生。

` [NIOServerCxn.Factory:0.0.0.0/0.0.0.0:2181:NIOServerCnxn$Factory@247] - Too many connections from /10.10.8.136 - max is 10

<!-- more -->

####解决
其实需要在zoo.cfg中加入maxClientCnxns=300，加完以后需要重启。问题解决。

####原因
我们线上有24台节点，但是这个参数竟然是使用默认的10，导致更多的客户端连上了zookeeper导致namenode的自带管理页无法连接到zookeeper，进而无法显示该页面。

这个教训告诉我不要盲目认为按照默认参数配置就没问题了，那是给小批量测试用的，需要根据实际情况采取相应配置，做运维不能刻舟求剑地认为默认没列出来的参数就不要去设置，需要自己去主动关注。等出了问题了别总怪东西不好用，硬件配置不够，分析问题要找症结。


如何监控zookeeper的其他指标，这里列出zoo.cfg的配置文件
```
dataDir = 数据存放路径

dataLogDir = 日志存放路径

clientPort = 客户端连接端口

clientPortAddress

tickTime= 整形 不能为0

maxClientCnxns= 整形 最大客户端连接数

minSessionTimeout= 整形

maxSessionTimeout= 整形

initLimit = 整形

syncLimit = 整形

electionAlg = 整形

peerType = observer | participant

server. sid= host:port | host:port:port  | host:port:port:type (type值 observer | participant)

group.gid = sid:sid (一个ID， 值是多个sid, 中间以:分割， 一个sid只能属于一个gid)

weight.sid=整形
```
可以看出还有至少2个参数是需要考虑的minSessionTimeout和maxSessionTimeout需要调优，得用JMX监控一段时间得出结论了。

同样的发现thrift也存在类似一连就断开的问题，下篇博文再作分析。




