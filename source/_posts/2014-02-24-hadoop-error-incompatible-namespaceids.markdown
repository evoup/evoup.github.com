---
layout: post
title: "hadoop报错：Incompatible namespaceIDs"
date: 2014-02-24 17:08
comments: true
categories: [hadoop]
---

今天在修改hadoop主机名重新格式化namenode之后，重新启动hadoop，发现datanode无法启动起来。

报错：HADOOP报错` Incompatible namespaceIDs `


查看报告发现没有启动一个datanode
<!-- more -->

```sh
$/u01/app/hadoop/bin/hadoop dfsadmin -report
Configured Capacity: 0 (0 KB)
Present Capacity: 0 (0 KB)
DFS Remaining: 0 (0 KB)
DFS Used: 0 (0 KB)
DFS Used%: �%
Under replicated blocks: 0
Blocks with corrupt replicas: 0
Missing blocks: 0

-------------------------------------------------
Datanodes available: 0 (0 total, 0 dead)
```

原来是要求datanode的VERSION文件和namenode的要一致

于是到namenode上看文件

```sh
[hadoop@mdn2 current]$more /u01/app/hadoopTmp/dfs/name/current/VERSION
#Mon Feb 24 16:48:12 CST 2014
namespaceID=1235115105
cTime=0
storageType=NAME_NODE
layoutVersion=-31
```

namespaceID为1235115105

到datanode里查看发现不存在


###解决方法两种任选其一：

1）在datanode的<dfs.data.dir>/current/VERSION中指定一个一模一样的namespaceID=1235115105，然后重启datanode

2）在格式化namenode的时候要清空/tmp目录下所有有关hadoop的目录，不论是namenode还是datanode所在的机器

####参考文章

http://blog.csdn.net/wanghai__/article/details/5752199
