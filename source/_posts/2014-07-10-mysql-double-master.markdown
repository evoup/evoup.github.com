---
layout: post
title: "mysql主主配置"
date: 2014-07-10 16:20
comments: true
categories: [mysql]
---
由于在项目中使用采用mysql的双master来实现关系型数据库的HA，以下配置的步骤记录一遍。

<!--more-->
###作业环境：
serverA:  IP:192.168.216.210  OS:CentOs6.4 64bit  DB:MySQL5.5.37

serverB:  IP:192.168.216.211  OS:CentOs6.4 64bit  DB:MySQL5.5.37

###Step1:同步授权
ServerA:
```sh
GRANT REPLICATION SLAVE,FILE ON *.* TO repuser@'192.168.216.211' IDENTIFIED BY 'reppass';
flush privileges;
$ sudo service mysql-server restart
```

ServerB:
```sh
GRANT REPLICATION SLAVE,FILE ON *.* TO repuser@'192.168.216.210' IDENTIFIED BY 'reppass';
flush privileges;
$ sudo service mysql-server restart
```

###Step2:配置参数
先找一下my.cnf的位置
```sh
$ mysql --verbose --help | grep -A 1 'Default options'
/etc/my.cnf /etc/mysql/my.cnf /usr/local/etc/my.cnf /usr/local/etc/mysql/my.cnf ~/.my.cnf
$ ls /etc/my.cnf /etc/mysql/my.cnf /usr/local/etc/my.cnf /usr/local/etc/mysql/my.cnf ~/.my.cnf
ls: /etc/my.cnf: No such file or directory
ls: /etc/mysql/my.cnf: No such file or directory
ls: /home/evoup/.my.cnf: No such file or directory
ls: /usr/local/etc/my.cnf: No such file or directory
ls: /usr/local/etc/mysql/my.cnf: No such file or directory
$ locate my-medium.cnf
/usr/local/share/mysql/my-medium.cnf
$ sudo cp /usr/local/share/mysql/my-medium.cnf /etc/my.cnf
```

开启二进制日志
接下来需要在ServerA的/etc/my.cnf中的mysqld上下文中添加配置
```
user=mysql
binlog-do-db=monitor1_db
binlog-ignore-db=mysql
replicate-do-db=monitor1_db
replicate-ignore-db=mysql
log-slave-update
slave-skip-errors=all
skip-name-resolve
sync_binlog=1
auto_increment_increment=2
auto_increment_offset=1
server-id=1
```
而在ServerB的/etc/my.cnf中
```
user=mysql
binlog-do-db=monitor1_db
binlog-ignore-db=mysql
replicate-do-db=monitor1_db
replicate-ignore-db=mysql
log-slave-update
slave-skip-errors=all
skip-name-resolve
sync_binlog=1
auto_increment_increment=2
auto_increment_offset=2
server-id=2
```

几点说明：
1) binlog-do-db和replicate-do-db是指出要同步的数据库，一般指定了数据库再其下创建的表就都能进行主主同步。<br>
2) auto_increment_offset一定要指定为1个奇数，1个偶数，为什么，估计是同步的机制，反正这官方的主主解决方案我感觉很奇葩,必须这么设置。<br>
3）同步的数据库要有上面指出的帐号（repuser）操作的权限。

###Step3:关联Master和Master
首先ServerA和ServerB要锁表，还有关闭slave同步
```
mysql> flush tables with read lock;
mysql> stop slave;
```
然后先查看ServerB的master状况
```
mysql>show master status;
+------------------+----------+--------------+------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+------------------+----------+--------------+------------------+
| mysql-bin.000019 |      107 | monitor1_db  | mysql            |
+------------------+----------+--------------+------------------+
```
在ServerA绑定好到ServerB的同步关系
```
mysql> change master to master_host='192.168.216.211',master_password='madcore',master_user='madcore',master_log_file='mysql-bin.000019',master_log_pos=107;
```

同样的，再查看ServerA的master状况
```
mysql> show master status;
+------------------+----------+--------------+------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+------------------+----------+--------------+------------------+
| mysql-bin.000018 |      23  | monitor1_db  | mysql            |
+------------------+----------+--------------+------------------+
```

再ServerB绑定好到ServerA的同步关系
```
mysql> change master to master_host='192.168.216.210',master_password='madcore',master_user='madcore',master_log_file='mysql-bin.000018',master_log_pos=23;
```

在ServerA和ServerB打开slave和解锁表
```
mysql> slave start;
mysql> unlock tables;
```
然后查看slave的同步状态
···
show slave status\G;
···
观察几个指标
Slave_IO_Running: Yes
Slave_SQL_Running: Yes
Seconds_Behind_Master:0

再看一下进程描述
```
mysql> show processlist;
```
一般上面几个指标是YES和0的话，这个不看都可以。

###补充：
在同步设置中遇到的问题，可以通过查看mysql日志的方法来解决。
查看mysql日志的位置的命令。
```sh
mysql> show variables like 'general_log_file';
```

