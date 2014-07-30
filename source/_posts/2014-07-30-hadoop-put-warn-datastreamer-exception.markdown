---
layout: post
title: "hadoop put文件报DataStreamer Exception异常"
date: 2014-07-30 17:05
comments: true
categories: [hadoop]
---
今天在一个hadoop节点上传测试文件的时候
```bash
$ bin/hadoop fs -put /home/hadoop/project/s3log.txt /yin_test/s3log
```
出现如下报错:
<!-- more -->
```
14/07/25 13:23:05 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where app
licable
14/07/25 13:23:06 WARN hdfs.DFSClient: DataStreamer Exception
org.apache.hadoop.ipc.RemoteException(java.io.IOException): File /yin_test/s3log/s3log.txt._COPYING_ could only be replicated to 0 nodes instead of minReplication (=1).  There are 0 datanode(s) running and no node(s) are excluded in this operation.
        at org.apache.hadoop.hdfs.server.blockmanagement.BlockManager.chooseTarget(BlockManager.java:1361)
        at org.apache.hadoop.hdfs.server.namenode.FSNamesystem.getAdditionalBlock(FSNamesystem.java:2362)
        at org.apache.hadoop.hdfs.server.namenode.NameNodeRpcServer.addBlock(NameNodeRpcServer.java:501)
        at org.apache.hadoop.hdfs.protocolPB.ClientNamenodeProtocolServerSideTranslatorPB.addBlock(ClientNamenodeProtocolServerSideTranslato
rPB.java:299)
        at org.apache.hadoop.hdfs.protocol.proto.ClientNamenodeProtocolProtos$ClientNamenodeProtocol$2.callBlockingMethod(ClientNamenodeProt
ocolProtos.java:44954)
        at org.apache.hadoop.ipc.ProtobufRpcEngine$Server$ProtoBufRpcInvoker.call(ProtobufRpcEngine.java:453)
        at org.apache.hadoop.ipc.RPC$Server.call(RPC.java:1002)
        at org.apache.hadoop.ipc.Server$Handler$1.run(Server.java:1752)
        at org.apache.hadoop.ipc.Server$Handler$1.run(Server.java:1748)
        at java.security.AccessController.doPrivileged(Native Method)
        at javax.security.auth.Subject.doAs(Subject.java:415)
        at org.apache.hadoop.security.UserGroupInformation.doAs(UserGroupInformation.java:1438)
        at org.apache.hadoop.ipc.Server$Handler.run(Server.java:1746)

        at org.apache.hadoop.ipc.Client.call(Client.java:1238)
        at org.apache.hadoop.ipc.ProtobufRpcEngine$Invoker.invoke(ProtobufRpcEngine.java:202)
        at com.sun.proxy.$Proxy9.addBlock(Unknown Source)
        at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
        at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:57)
        at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
        at java.lang.reflect.Method.invoke(Method.java:606)
        at org.apache.hadoop.io.retry.RetryInvocationHandler.invokeMethod(RetryInvocationHandler.java:164)
        at org.apache.hadoop.io.retry.RetryInvocationHandler.invoke(RetryInvocationHandler.java:83)
        at com.sun.proxy.$Proxy9.addBlock(Unknown Source)
```
已经把防火墙关了，经过一番查询研究，原来是dfs.namenode.name.dir和dfs.datanode.data.dir再重新格式化后要被清空的原因。删除这2个目录下的文件，然后重新格式化，再次put恢复正常。
