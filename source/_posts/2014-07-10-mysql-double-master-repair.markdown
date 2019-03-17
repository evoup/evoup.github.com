---
layout: post
title: "mysql复制同步修复工具"
date: 2014-07-10 16:44
comments: true
categories: [mysql]
---

采用mysql双主同步这种架构时要格外注意数据同步的问题，如果没有解决好不同步的问题，那就像《网站运维：保持数据实时的秘技》中提到的，简直就是期待数据不同步。所以对于实际使用过程中还是要紧密监控，对于数据不同步的状况，需要及时报警和处理，不然就等着数据异常吧！下文是主从同步的修复，主主的修复，只要把另外一个主看成从，互相修一下就可以了，见下文：

<!--more-->

###恢复主主的工具

```bash
> wget http://www.percona.com/redir/downloads/percona-toolkit/LATEST/tarball/percona-toolkit-2.2.10.tar.gz 
> tar xzf percona-toolkit-2.2.6.tar.gz
> cd percona-toolkit-2.2.6
> perl Makefile.PL
Warning: prerequisite DBD::mysql 3 not found.
Warning: prerequisite DBI 1.46 not found.
Writing Makefile for percona-toolkit

> sudo cpan YAML
> sudo cpan DBD::mysql
```

出现询问是否yes，一路选择yes


遇到测试的时候出现
` FAILED--Further testing stopped: ERROR: Access denied for user 'root'@'localhost' (using password: NO) `

进入mysql修改一下默认密码为空

```mysql
GRANT ALL PRIVILEGES ON *.* TO root@'localhost' IDENTIFIED BY '';
flush privileges;
```

回来继续

```bash
sudo cpan DBD::mysql
make
sudo make install
```

###检查完整性

```bash
[yinjia@hm15hadoop01 percona-toolkit-2.2.6]>sudo pt-table-checksum  --recursion-method=processlist
```

` Cannot connect to h=10.10.8.45 `

` Diffs cannot be detected because no slaves were found.  Please read the --recursion-method documentation for information. `


报错，在主上运行

```
mysql> show slave hosts;
+-----------+------+------+-----------+
| Server_id | Host | Port | Master_id |
+-----------+------+------+-----------+
|         2 |      | 3306 |         1 |
+-----------+------+------+-----------+
1 row in set (0.00 sec)
```

发现是host没有所以导致脚本会报错

到slave上，my.cnf中添加report_host=10.10.8.45 #设置成本机IP，然后重启slave
回到主上

```
mysql> show slave hosts;
+-----------+------------+------+-----------+
| Server_id | Host       | Port | Master_id |
+-----------+------------+------+-----------+
|         2 | 10.10.8.45 | 3306 |         1 |
+-----------+------------+------+-----------+
1 row in set (0.00 sec)

>sudo pt-table-checksum --recursion-method=processlist
Password:
Cannot connect to h=10.10.8.45
Diffs cannot be detected because no slaves were found.  Please read the --recursion-method documentation for information.
```

问题依旧,修改shell

```bash
[yinjia@hm15hadoop01 percona-toolkit-2.2.6]>pt-table-checksum --recursion-method=hosts --no-check-binlog-format --nocheck-replication-filters --replicate=monitor1_db.checksums --databases=monitor1_db --tables=httplog_status h=10.10.8.45,u=madcore,p=madcore,P=3306
            TS ERRORS  DIFFS     ROWS  CHUNKS SKIPPED    TIME TABLE
06-30T13:33:50      0      1     5415       4       0   0.377 monitor1_db.httplog_status
```

看到DIFFS1，即为有1行差异

```bash
[yinjia@hm15hadoop02 ~/software]>pt-table-checksum --recursion-method=hosts --no-check-binlog-format --nocheck-replication-filters --replicate=monitor1_db.checksums --databases=monitor1_db --tables=httplog_status h=10.10.8.44,u=madcore,p=madcore,P=3306
            TS ERRORS  DIFFS     ROWS  CHUNKS SKIPPED    TIME TABLE
06-30T13:34:58      0      1     5395       4       0   0.371 monitor1_db.httplog_status
```

可以看到现在DIFF是1，证明有差异。所以要同步，怎么同步，使用pt-table-sync进行修复

###修复同步

```bash
pt-table-sync --replicate=monitor1_db.checksums h=10.10.8.44,u=madcore,p=madcore h=10.10.8.45,u=madcore,p=madcore --print --execute
```

分别是主的IP，从的IP。针对两边分别修复，最后再看一下pt-table-checksum 发现DIFF为0证明数据同步成功。


###注意事项
同步修复后，记录可能发生乱序，比较好的做法是加timestamp字段，把写入的时间戳记载下来，然后select的时候采用ORDER BY进行排序，这样就不会乱序了。



