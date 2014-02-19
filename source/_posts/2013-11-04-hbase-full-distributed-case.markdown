---
layout: post
title: "HBASE完全分布式搭建(VMware版)"
date: 2013-11-04 16:28
comments: true
categories: hadoop 
---
接上一篇<a href="http://evoupsight.com/blog/2013/11/04/hadoop-full-distributed-case/">《HADOOP完全分布式搭建(VMware版)》</a>


参考 http://www.cnblogs.com/flyoung2008/archive/2011/12/02/2272761.html

<!-- more -->

```bash
cd /u01/app
tar xzf hbase-0.90.6.tgz
ln -s hbase-0.90.6 hbase
cd hbase/conf
```

编辑hbase-env.sh
```bash
export HBASE_OPTS="-ea -XX:+UseConcMarkSweepGC -XX:+CMSIncrementalMode"
export JAVA_HOME=/usr/java/jdk1.6.0_29
export HBASE_MANAGES_ZK=true
```

注意HADOOP_HOME和HBASE_HOME已经在~/.profile中指定，不需要再设置了。

--------------

(补充：2014-2-19 发现这么写还不能加载，放到~/.bashrc中才对，见下)
```bash
export HADOOP_HOME=/u01/app/hadoop
```

--------------

编辑hbase-site.xml
```xml
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<!--
/**
 * Copyright 2010 The Apache Software Foundation
 *
 * Licensed to the Apache Software Foundation (ASF) under one
 * or more contributor license agreements.  See the NOTICE file
 * distributed with this work for additional information
 * regarding copyright ownership.  The ASF licenses this file
 * to you under the Apache License, Version 2.0 (the
 * "License"); you may not use this file except in compliance
 * with the License.  You may obtain a copy of the License at
 *
 *     http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
-->
<configuration>
    <property>
        <name>hbase.rootdir</name>
        <value>hdfs://mdn2.net:9024/hbase</value>
    </property>
    <property>
        <name>hbase.cluster.distributed</name>
        <value>true</value>
    </property>
    <property>
        <name>hbase.master</name>
        <value>mdn2.net:60000</value>
    </property>
    <property>
        <name>hbase.zookeeper.quorum</name>
        <value>mdn2.net,mdn2_datanode1.net,mdn2_datanode2.net</value>
    </property>
</configuration>
```

注意点：
 1.其中首先需要注意hdfs://mdn2.net:9024/hbase这里，必须与你的Hadoop集群的core-site.xml文件配置保持完全一致才行，如果你Hadoop的hdfs使用了其它端口，请在这里也修改。再者就是Hbase该项并不识别机器IP，只能使用机器hostname才可行，即若使用IP是会抛出java错误。
 2.hbase.zookeeper.quorum 的个数必须是奇数。

修改regionservers文件（同hadoop的slaves文件）
```bash
mdn2_datanode1.net
mdn2_datanode2.net
```

然后分发到各点，就可以启动了。

```bash
bin/start hbase
bin/hbase shell
```

报错
ERROR: org.apache.hadoop.hbase.ZooKeeperConnectionException: HBase is able to connect to ZooKeeper but the connection closes immediately. This could be a sign that the server has too many connections (30 is the default). Consider inspecting your ZK server logs for that error and then make sure you are reusing HBaseConfiguration as often as you can. See HTable's javadoc for more information.
看来不要使用hbase托管的zookeeper转而再装一个试试。

```bash
wget http://mirrors.tuna.tsinghua.edu.cn/apache/zookeeper/zookeeper-3.4.4/zookeeper-3.4.4.tar.gz
```

编辑~/.profile,加入关于zk环境变量的设置

```bash
export ZOOKEEPER_HOME="/u01/app/zookeeper/"
PATH=$ZOOKEEPER_HOME/bin:$PATH
export PATH
cd /u01/app/zookeeper/conf
cp zoo_sample.cfg zoo.cfg
cd ../bin
./zkServer.sh start
```

最后重启整个hadoop/hbase搞定，jps看下跑的进程。收工。

![Alt text](/images/evoup/hbase_vmware.png)


