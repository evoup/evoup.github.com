---
layout: post
title: "导入mysql的几种方法"
date: 2012-09-16 14:16
comments: true
categories: [mysql] 
---

mysql这玩意官方导入的方法我初步估算了一下，有5种。这里不涉及使用第三方软件导入的分析（如phpmyadmin）。且听下文点评各个方案要注意的部分，有错之处还请不吝赐教。

<!-- more -->
####1）mysql加管道符导入
这是最基本的方法,语法见下：
```sh
mysql -uuser -p dbname < import.sql 
```
这种方案导入的时候要注意编码的问题，通过加入–default-character-set = utf8来解决，此外好像对csv和纯文本文件的导入支持比较差。要强制导入成功可以用--force参数。


####2）load data infile
这个需要在mysql的shell中跑，语法见下：
```sh
mysql> use dbname;set names utf8;load data infile "/path/to/import.csv" into table tbname(field1,field2)
```
这种方案默认的列分隔符为\t而行分隔符为\n，也就是说文本导入默认是unix换行格式的。而空数据的导入默认是认为\N导入之后就是NULL了。除此之外必须指定表名字，也就是说你有多个分表要导入就要写很多语句或者写一个shell遍历这么多的表，效率是可想而知的慢。


####3）source
这个同样是需要在mysql的shell中跑，语法见下：
```sh
mysql> use dbname;set names utf8;source import.sql
```
这种方案我没怎么试过，据说会出现导入过程中卡死，退出表损坏的状况，不推荐大胆尝试。


####4）mysql -e
这个方法其实就是执行若干sql，语法见下：
```sh
mysql -uuser -p -e "use dbname;set names utf8;insert into tbname(field1,field2) values('foo','bar');"
```
所以这其实称不上是什么方法，前面的方法2和方法3都可以偷梁换柱啦：）

####5）mysqlimport
这是mysql官方安装包提供的工具了，使用方法：
```sh
mysqlimport -uuser -p -h127.0.0.1 dbname importdir/* --field-terminated-by=',' --filelds-optionally-enclosed-by='"' --local --compress --use-thread=12
```
以上这条语句代表把本地文件夹importdir下的所有准备导入的文本，按照以逗号为字段分割符，双引号为行分隔符，以压缩的方式，开12个线程同时导入到位于127.0.0.1上mysql服务器的dbname库中。
可见这种方案可以多表并发导入，不需要写多条语句就可以完成任务，推荐使用,另外提一句，上面的语句只需要用php的fputcsv函数并指定文件资源和写入行字符串这2个参数就可以了。
最后，如果使用mysqldump最好不要用-T的方式导出，因为据说无法使用set unique_checks、set autocommit等实现加速导入。
