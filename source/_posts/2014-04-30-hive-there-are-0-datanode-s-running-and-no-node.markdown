---
layout: post
title: "hive There are 0 datanode(s) running and no node"
date: 2014-04-30 17:38
comments: true
categories: [hadoop,hive]
---

hive查询报错` c
ould only be replicated to 0 nodes instead of minReplication (=1).  There are 0 datanode(s) running and no node(s) are excluded in this operation `

<!-- more -->

```
hive> select count(*) from httplog_2014_05_26_2017;
Total MapReduce jobs = 1
Launching Job 1 out of 1
Number of reduce tasks determined at compile time: 1
In order to change the average load for a reducer (in bytes):
  set hive.exec.reducers.bytes.per.reducer=<number>
In order to limit the maximum number of reducers:
  set hive.exec.reducers.max=<number>
In order to set a constant number of reducers:
  set mapred.reduce.tasks=<number>
org.apache.hadoop.ipc.RemoteException(java.io.IOException): File /tmp/hive-hadoop/hive_2014-07-15_10-27-16_252_580204120341469236-1/-mr-10003/7a56500c-c6e3-4f8c-a32b-053e9fe8eec2 c
ould only be replicated to 0 nodes instead of minReplication (=1).  There are 0 datanode(s) running and no node(s) are excluded in this operation.
        at org.apache.hadoop.hdfs.server.blockmanagement.BlockManager.chooseTarget(BlockManager.java:1340)
        at org.apache.hadoop.hdfs.server.namenode.FSNamesystem.getAdditionalBlock(FSNamesystem.java:2296)
        at org.apache.hadoop.hdfs.server.namenode.NameNodeRpcServer.addBlock(NameNodeRpcServer.java:501)
        at org.apache.hadoop.hdfs.protocolPB.ClientNamenodeProtocolServerSideTranslatorPB.addBlock(ClientNamenodeProtocolServerSideTranslatorPB.java:299)
        at org.apache.hadoop.hdfs.protocol.proto.ClientNamenodeProtocolProtos$ClientNamenodeProtocol$2.callBlockingMethod(ClientNamenodeProtocolProtos.java:44954)
        at org.apache.hadoop.ipc.ProtobufRpcEngine$Server$ProtoBufRpcInvoker.call(ProtobufRpcEngine.java:453)
        at org.apache.hadoop.ipc.RPC$Server.call(RPC.java:1002)
        at org.apache.hadoop.ipc.Server$Handler$1.run(Server.java:1752)
        at org.apache.hadoop.ipc.Server$Handler$1.run(Server.java:1748)
```

需要清理目录，其中hadoop为当前用户的名字
```bash
cd /tmp/hadoop
rm -rf *
```

然后重启hadoop和hive，恢复正常，原因是hadoop的namespaceIDs有异常。



