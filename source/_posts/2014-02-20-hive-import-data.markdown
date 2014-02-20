---
layout: post
title: "hive初试--导入数据和查询"
date: 2014-02-20 10:50
comments: true
categories: hadoop 
---

hive虽然是基于hadoop的map/reduce进行云计算，但是自身需要依赖一个元数据表，要么是derby，要么是mysql，相同点总归是要先导入数据，然后才能进行处理。其原理是把结构化的数据文件映射为一张数据库表，然后将SQL语句转化为MapReduce任务进行运行，以绕过专门开发MapReduce这样一个逆向思维的产物。

Hive不可以改写、插入和删除数据，换句话说hive完全就是用来进行计算的。

Hive的数据是存在hdfs上的,所以数据导入之后除了元数据之外，还有另一份本体数据（通常比较大的）存在hdfs上。

有了基础概念之后，开始正题了。

<!-- more -->
###任务描述
本次目的，是把以下二维表导入到hive中后，根据编号查询对应的单词。

###过程描述
假设有这样一个文件

cat test.txt
1	hello
2	world
3	test
4	case
(vim党注意：如果你已经把tab键映射为4个空格，那么请进入插入模式后在数字后ctrl+v,然后按下<tab>键，再输入单词，否则无法完成制表符的键入，数据导入失败。)

启动hive建表:
```sh
hive>  CREATE EXTERNAL TABLE MYTEST(id INT, name STRING) 
> COMMENT 'this is a test'
> ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
> STORED AS TEXTFILE
>LOCATION '/data/test';
OK
```
注意这一步要求原本的hdfs目录下没有/data/test文件夹，如果有的话，hive是要报错的。
还有存储格式有三种textfile、rcfile和sequencefile。其中多数情况用textfile就可以了，如果要压缩，可以考虑后两者。

进入hadoop，开始导入
```sh
/bin/hadoop fs -put test.txt /data/test
```

回到hive，用简单的HQL查询语句查询id为4的记录
```sh
hive> select * from mytest where id = 4;
Total MapReduce jobs = 1
Launching Job 1 out of 1
Number of reduce tasks is set to 0 since there's no reduce operator
Starting Job = job_201402191826_0007, Tracking URL = http://mdn2.net:50030/jobdetails.jsp?jobid=job_201402191826_0007
Kill Command = /u01/app/hadoop/bin/../bin/hadoop job  -Dmapred.job.tracker=mdn2.net:9025 -kill job_201402191826_0007
2014-02-20 00:16:34,842 Stage-1 map = 0%,  reduce = 0%
2014-02-20 00:16:40,889 Stage-1 map = 100%,  reduce = 0%
2014-02-20 00:16:46,936 Stage-1 map = 100%,  reduce = 100%
Ended Job = job_201402191826_0007
OK
4       case
Time taken: 21.36 seconds

```
hive查询一次需要21秒?没错，这就是MapReduce查询的特点了，换做mysql的话这样查询一次应该是<1秒的。好啦，收工。
