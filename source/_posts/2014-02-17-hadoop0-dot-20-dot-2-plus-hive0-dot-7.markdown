---
layout: post
title: "centos5.8 hadoop0.20.2+hive0.7 installation"
date: 2014-02-17 15:38
comments: true
categories: [hadoop] 
---

hadoop0.20.203下hive0.7安装

###什么是hive

> hive是基于Hadoop的一个数据仓库工具，可以将结构化的数据文件映射为一张数据库表，并提供简单的sql查询功能，可以将sql语句转换为MapReduce任务进行运行。 其优点是学习成本低，可以通过类SQL语句快速实现简单的MapReduce统计，不必开发专门的MapReduce应用，十分适合数据仓库的统计分析。

<!-- more -->

###hive的安装

先提一下，之前我的hadoop安装在/u01/app/hadoop目录下，同样的我们下载到改目录下，然后开始安装
注意我的hadoop版本为hadooop-0.20.203.0.tgz，与此匹配的版本为hive0.7.0
```sh
cd /u01/app
wget http://archive.apache.org/dist/hive/hive-0.7.1/hive-0.7.1-bin.tar.gz
tar xzf hive-0.7.1-bin.tar.gz
ln -s hive-0.7.1-bin.tar.gz hive
```

####环境变量设置
在~/.profile中加入
```sh
export HIVE_HOME=/u01/app/hive
export HIVE_CONF_DIR=/u01/app/hive/conf
```
//在系统中指出hive的配置文件所在
```sh
export PATH=$HIVE_HOME/bin:PATH
```
//这个实现输入hive，hive service就会自动相应，而不用输入hive所在的绝对路径。
```sh
export HIVE_LIB=$HIVE_HOME/lib
```

记得用source让profile生效
```sh
source ~/.profile
```

然后是进行hive配置文件的配置
```sh
cd /u01/app/hive/conf
cp hive-env.sh.template hive-env.sh
vim hive-env.sh
```



####安装依赖软件
发现有2种安装方式，一种是derby,另一种是mysql，这里先介绍mysql方式

> 关于什么是derby: 这是一个apache DB的子项目，是一个完全用java实现的开源关系型数据库。这里就不使用了，我们采用mysql。


####安装mysql
```sh
//卸载老版本的mysql软件包
yum remove mysql mysql-*
//安装mysql5.5的源
rpm -Uvh http://repo.webtatic.com/yum/centos/5/latest.rpm
//安装MySQL客户端的支持包
yum install libmysqlclient15 --enablerepo=webtatic
//安装MySQL 5.5的客户端和服务端
yum install mysql55 mysql55-server --enablerepo=webtatic
//启动MySQL系统服务，更新数据库
/etc/init.d/mysqld restart
mysql_upgrade
```

####修改mysql用户密码
```sh
# mysql -u root mysql   //默认的没有密码直接进去的
mysql>use mysql;
mysql>desc user;
mysql> GRANT ALL PRIVILEGES ON *.* TO root@"%" IDENTIFIED BY "root";　　//为root添加远程连接的能力。
mysql>update user set Password = password('xxxxxx') where User='root';
mysql>select Host,User,Password  from user where User='root';
mysql>flush privileges;
mysql>exit
```
####设置mysql为开机自动启动
```sh
sudo /sbin/chkconfig --add mysqld
sudo /sbin/chkconfig mysqld on
```

####开始配置
在conf目录下创建hive-site.xml

创建hive数据库给hive做元数据表
```sh
create database hive;
grant all privileges on *.* to hive@localhost identified by 'hive';
flush privileges;
```

运行hive
```sh
cd /u01/app/hive
/bin/hive
Exception in thread "main" java.lang.NoClassDefFoundError: jline/ArgumentCompletor$ArgumentDelimiter
        at java.lang.Class.forName0(Native Method)
        at java.lang.Class.forName(Class.java:247)
        at org.apache.hadoop.util.RunJar.main(RunJar.java:149)
Caused by: java.lang.ClassNotFoundException: jline.ArgumentCompletor$ArgumentDelimiter
        at java.net.URLClassLoader$1.run(URLClassLoader.java:202)
        at java.security.AccessController.doPrivileged(Native Method)
        at java.net.URLClassLoader.findClass(URLClassLoader.java:190)
        at java.lang.ClassLoader.loadClass(ClassLoader.java:306)
        at java.lang.ClassLoader.loadClass(ClassLoader.java:247)
        ... 3 more
```
这个只需要把hive/lib下的jline-0.9.94.jar复制到$HADOOP/lib下即可。

再次启动
```sh
bin/hive
Exception in thread "main" java.lang.NoClassDefFoundError: org/apache/hadoop/hive/conf/HiveConf
        at java.lang.Class.forName0(Native Method)
        at java.lang.Class.forName(Class.java:247)
        at org.apache.hadoop.util.RunJar.main(RunJar.java:149)
Caused by: java.lang.ClassNotFoundException: org.apache.hadoop.hive.conf.HiveConf
        at java.net.URLClassLoader$1.run(URLClassLoader.java:202)
        at java.security.AccessController.doPrivileged(Native Method)
        at java.net.URLClassLoader.findClass(URLClassLoader.java:190)
        at java.lang.ClassLoader.loadClass(ClassLoader.java:306)
        at java.lang.ClassLoader.loadClass(ClassLoader.java:247)
```

需要修改$HADOOP/conf/的hadoop-env.sh
中的
```sh
export HADOOP_CLASSPATH=$HBASE_HOME/hbase-0.90.3.jar:$HBASE_HOME:$HBASE_HOME/lib/zookeeper-3.2.2.jar:$HBASE_HOME/conf
```

改成

```sh
export HADOOP_CLASSPATH=$HADOOP_CLASSPATH:$HBASE_HOME/hbase-0.90.3.jar:$HBASE_HOME:$HBASE_HOME/lib/zookeeper-3.2.2.jar:$HBASE_HOME/conf
```

然后可以启动hive了
```sh
bin/hive
WARNING: org.apache.hadoop.metrics.jvm.EventCounter is deprecated. Please use org.apache.hadoop.log.metrics.EventCounter in all the log4j.properties files.
Hive history file=/tmp/hadoop/hive_job_log_hadoop_201402170220_1889385824.txt
hive>
```
有警告，估计是jdk我用的1.7导致的，可以先不管，接下来可以试试hive的操作了:)


###参考文章

http://blog.163.com/huang_zhong_yuan/blog/static/174975283201181371146365/

http://hi.baidu.com/allense7en/item/db8e5b4fb177aae81e19bcb4

