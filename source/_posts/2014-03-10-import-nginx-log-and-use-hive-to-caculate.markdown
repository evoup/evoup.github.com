---
layout: post
title: "导入nginx日志并采用hive进行统计"
date: 2014-03-10 16:07
comments: true
categories: hadoop
---

公司的日志本来是存成gzip或bz2格式直接导入到hdfs里去然后用程序直接mr的，速度是比较慢的。领导要求采用hive来计算，而在hive里建textfile表的这种方式，textfile是无法进行并行计算的，而且gzip和bz2做mr的速度是很慢的。所以准备采取先导入hdfs和textfile表，然后再转换为rcfile格式的表的策略。实际试验下来，如果一开始转换为文本或者是lzo格式，而不是采用gzip或bz2的格式的textfile的表，再转换为rcfile的方式会快很多，mr的速度也是比较快的。

下面描述一下过程
<!-- more -->


把全部日志上通过scp等方式传到服务器之后，要做的是先建一个textfile的表
```sh
create external table nginxlog (ipaddress string, ...更多字段省略) COMMENT 'nginx log' ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' stored as textfile    location '/data/log';
```

把得到所有nginx日志，然后用lzop先压缩好，然后传到hdfs上去。
```sh
$ tar xjf server01-20140131.txt.bz2
$ tar xjf server02-20140131.txt.bz2
$ lzop server01-20140131.txt 
$ lzop server02-20140131.txt 
$ server01-20140131.txt.lzo server02-20140131.txt.lzo
$ /u01/app/hadoop fs -put server01-20140131.txt.lzo /data/log/server01-20140131.txt.lzo
$ /u01/app/hadoop fs -put server01-20140131.txt.lzo /data/log/server02-20140131.txt.lzo
$/u01/app/hadoop/bin/hadoop fs -ls /data/log/
Found 2 items
-rw-r--r--   1 hadoop supergroup  364459530 2014-03-07 18:10 /data/log/server01-20140131.txt.lzo
-rw-r--r--   1 hadoop supergroup  364459530 2014-03-10 13:31 /data/log/server02-20140201.txt.lzo
```

然后马上就可以查询了
```
hive> select count(*) from nginxlog;
Total MapReduce jobs = 1
Launching Job 1 out of 1
Number of reduce tasks determined at compile time: 1
In order to change the average load for a reducer (in bytes):
  set hive.exec.reducers.bytes.per.reducer=<number>
In order to limit the maximum number of reducers:
  set hive.exec.reducers.max=<number>
In order to set a constant number of reducers:
  set mapred.reduce.tasks=<number>
Starting Job = job_201403101051_0004, Tracking URL = http://mdn2.net:50030/jobdetails.jsp?jobid=job_201403101051_0004
Kill Command = /u01/app/hadoop/bin/../bin/hadoop job  -Dmapred.job.tracker=mdn2.net:9025 -kill job_201403101051_0004
2014-03-10 14:17:41,179 Stage-1 map = 0%,  reduce = 0%
2014-03-10 14:18:20,815 Stage-1 map = 50%,  reduce = 0%
2014-03-10 14:18:32,927 Stage-1 map = 100%,  reduce = 0%
2014-03-10 14:18:38,971 Stage-1 map = 100%,  reduce = 17%
2014-03-10 14:18:41,991 Stage-1 map = 100%,  reduce = 100%
Ended Job = job_201403101051_0004
OK
2492916
Time taken: 75.321 seconds
```

这么做也是可以使用hive的，但是速度还是比较慢。于是可以再创建一个rcfile格式的表，然后再查询
```
bin/hive> create external table nginxlog2 (ipaddress string, ...,更多字段) COMMENT 'nginx log rcfile format' ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' stored as RCFile    location '/data/log2';
```

然后用
```
hive> SET hive.exec.compress.output=true;
hive> SET io.rcfile.compression.type=BLOCK;
hive> insert overwrite table nginxlog2 select * from nginxlog;
Total MapReduce jobs = 2
Launching Job 1 out of 2
Number of reduce tasks is set to 0 since there's no reduce operator
Starting Job = job_201403101051_0007, Tracking URL = http://mdn2.net:50030/jobdetails.jsp?jobid=job_201403101051_0007
Kill Command = /u01/app/hadoop/bin/../bin/hadoop job  -Dmapred.job.tracker=mdn2.net:9025 -kill job_201403101051_0007
2014-03-10 15:20:20,959 Stage-1 map = 0%,  reduce = 0%
2014-03-10 15:21:21,267 Stage-1 map = 0%,  reduce = 0%
2014-03-10 15:22:21,627 Stage-1 map = 0%,  reduce = 0%
2014-03-10 15:23:22,320 Stage-1 map = 0%,  reduce = 0%
2014-03-10 15:23:36,542 Stage-1 map = 100%,  reduce = 0%
2014-03-10 15:23:42,665 Stage-1 map = 100%,  reduce = 100%
Ended Job = job_201403101051_0007
Ended Job = -1308159129, job is filtered out (removed at runtime).
Moving data to: hdfs://mdn2.net:9024/tmp/hive-hadoop/hive_2014-03-10_15-20-15_076_2561493179927538497/-ext-10000
Loading data to table default.nginxlog2
Deleted hdfs://mdn2.net:9024/data/log2
Table default.nginxlog2 stats: [num_partitions: 0, num_files: 0, num_rows: 0, total_size: 0]
2492916 Rows loaded to nginxlog2
OK
Time taken: 209.088 seconds
```

然后再次select，对比一下时间
```
hive> select count(*) from nginxlog2;
Total MapReduce jobs = 1
Launching Job 1 out of 1
Number of reduce tasks determined at compile time: 1
In order to change the average load for a reducer (in bytes):
  set hive.exec.reducers.bytes.per.reducer=<number>
In order to limit the maximum number of reducers:
  set hive.exec.reducers.max=<number>
In order to set a constant number of reducers:
  set mapred.reduce.tasks=<number>
Starting Job = job_201403101051_0008, Tracking URL = http://mdn2.net:50030/jobdetails.jsp?jobid=job_201403101051_0008
Kill Command = /u01/app/hadoop/bin/../bin/hadoop job  -Dmapred.job.tracker=mdn2.net:9025 -kill job_201403101051_0008
2014-03-10 15:26:21,984 Stage-1 map = 0%,  reduce = 0%
2014-03-10 15:26:31,031 Stage-1 map = 33%,  reduce = 0%
2014-03-10 15:26:43,107 Stage-1 map = 67%,  reduce = 0%
2014-03-10 15:26:49,140 Stage-1 map = 67%,  reduce = 17%
2014-03-10 15:26:52,153 Stage-1 map = 100%,  reduce = 22%
2014-03-10 15:27:04,225 Stage-1 map = 100%,  reduce = 100%
Ended Job = job_201403101051_0008
OK
2492916
Time taken: 55.656 seconds
```
我这里是2个节点， 55.656s，相比textfile的75.321s，rcfile的有20秒的优势，当然并行计算的节点越多，时间就越省。

这里要补充一下：不通过本地导入的方式直接导入rcfile的原因，是因为textfile格式才支持从本地导入，sequencefile和rcfile均不支持，所以只能先搞一个表再复制。如果用textfile加gzip或bz2的表再复制到rcfile的表，时间会很长；而用textfile+lzo的表再复制到rcfile的表，时间比较短。lzo相对gzip或bz2压缩速度快但是相对压缩比没有优势，然而再转为rcfile格式mr会很快，这样hive查询就很快。


