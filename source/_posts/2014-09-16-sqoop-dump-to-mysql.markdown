---
layout: post
title: "sqoop导出数据到mysql"
date: 2014-09-16 18:30
comments: true
categories: [hive] 
---

现为apache项目的sqoop是原来是一款hadoop第三方开源模块，通过使用sqoop可以使进行hadoop(hive)的数据与传统关系型数据库（mysql、pgsql）交互。可以把关系型数据库导入hdfs，也可以把hdfs的数据导入关系型数据库。这里我用实例记录的方式讲一下怎么通过它把hive中的数据导入到mysql结果表中。
<!-- more -->
####安装
yum install sqoop-noarch sqoop-metastore.noarch

####建表
首先是hive的表结构
```sql
use db_work;
CREATE  TABLE table_work(
  item string,
  value string)
PARTITIONED BY (
  dt int,
  tm int)
ROW FORMAT DELIMITED
  FIELDS TERMINATED BY '\t'
STORED AS INPUTFORMAT
  'org.apache.hadoop.mapred.TextInputFormat'
OUTPUTFORMAT
  'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION
  'hdfs://db_work/hive/warehouse/db_work.db/table_work'
```

注意这里使用\t作为字段分隔符，与之后的sqoop保持一致。

mysql表的格式
```sql
use testdb;
CREATE TABLE `result` (
  `item` mediumtext,
  `value` mediumtext
) ENGINE=MyISAM DEFAULT CHARSET=utf8;
```
注意sqoop导入的字段如果过长，会报错` com.mysql.jdbc.MysqlDataTruncation: Data truncation: Data too long for column ` ，这里使用mediumtext解决这个问题。

####导入
导入语句
```bash
sqoop export -m8 --connect 'jdbc:mysql://'"${mysql_host}"':3306/testdb?useUnicode=true&characterEncoding=utf8' --username ${mysql_user} --password ${mysqlpass} --table result --export-dir /hive/warehouse/db_work.db/table_work/dt=20140916/tm=1213 --input-fields-terminated-by '\t' --input-null-string '\\N' --input-null-non-string '\\N'
```
以上语句可以正确地把名为table_work的hive表，以\t为列分隔符，\N为空数据，导入到名为result的mysql表



####参考文章
《sqoop 从 hive 导到mysql遇到的问题》
http://ju.outofmemory.cn/entry/44398

《Sqoop1:exportdatatomysql》
http://www.haogongju.net/art/2639227

