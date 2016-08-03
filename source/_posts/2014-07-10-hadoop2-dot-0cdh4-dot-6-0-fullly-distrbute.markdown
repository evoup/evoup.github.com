---
layout: post
title: "hadoop2.0cdh4.6.0完全分布式安装"
date: 2014-07-10 15:57
comments: true
categories: [hadoop,hbase,hive,zookeeper]
---
hadoop2.0 cdh4安装（完全分布式）

<!-- more -->

vmware版本8.0.4 build-744019

首先规划3台虚拟机

```bash
 ,'''''''''''''''''''''':'''''''''''''''''':''''''''''''''''''''''''''''''''''''''''''''|
 |        usage         |        IP        |                  Hostname                  |
 |                      |                  |                                            |
 |''''''''''''''''''''''|''''''''''''''''''|''''''''''''''''''''''''''''''''''''''''''''|
 | namenode1,datanode1  | 192.168.216.183  |    mdn3namenode1.net,mdn3datanode1.net     |
 |                      |                  |                                            |
 |''''''''''''''''''''''|''''''''''''''''''|''''''''''''''''''''''''''''''''''''''''''''|
 | namenode2,datanode2  | 192.168.216.184  |    mdn3namenode2.net,mdn3datanode2.net     |
 |                      |                  |                                            |
 |''''''''''''''''''''''|''''''''''''''''''|''''''''''''''''''''''''''''''''''''''''''''|
 | datanode2,nfs server | 192.168.216.185  |    mdn3datanode3.net,mdn3nfsserver.net     |
 |                      |                  |                                            |
  '''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''
```

###准备工作

先安装JDK1.6 linux:先把已经安装的openjdk卸载,安装sun jdk1.6,去oracle下载 （j2se就够了）

```bash
$ rpm -qa | grep jdk
java-1.6.0-openjdk-1.6.0.0-1.28.1.10.9.el5_8
$ sudo rpm -e java-1.6.0-openjdk-1.6.0.0-1.28.1.10.9.el5_8
$ sudo chmod +x jdk-6u45-linux-x64-rpm.bin
$ sudo ./jdk-6u45-linux-x64-rpm.bin
```

hadoop所有操作都是用hadoop帐号，下面添加（如果已经创建了帐号无须添加）

```bash
$ groupadd hadoop
$ useradd -r -g hadoop -d /home/hadoop -m -s /bin/bash hadoop

$ mkdir -p /home/hadoop
$ chgrp -R hadoop /home/hadoop
$ chown -R hadoop /home/hadoop
```

环境变量(在centos里不管编辑~/.profile还是~/.bash_profile都不能加载环境变量，正确的应该是在~/.bashrc中，而如果是root用户，应该可以直接在/etc/profile中编辑)

```bash
$ vi ~/.bashrc 
export HADOOP_HOME="/usr/local/hadoop"
export JAVA_HOME="/usr/java/jdk1.6.0_45"
export PATH=$PATH:$JAVA_HOME/bin
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
```


切换到hadoop帐号，进行免密码的ssh登录设置

```bash
$ su hadoop
$ ssh-keygen -t dsa -P '' -f ~/.ssh/id_dsa
$ cat ~/.ssh/id_dsa.pub >> ~/.ssh/authorized_keys
$ chmod 600 ~/.ssh/authorized_keys
```


给出我的hadoop/hbase版本

```bash
Name        : hadoop-hdfs-namenode
Arch        : x86_64
Version     : 2.0.0+1554
Release     : 1.cdh4.6.0.p0.16.el6

Name        : hbase-master
Arch        : x86_64
Version     : 0.94.15+86
Release     : 1.cdh4.6.0.p0.14.el6
```

然后是cdh的软件下载url
http://archive.cloudera.com/cdh4
这个路径下有很多的软件。

下载cdh4.6的几个包安装

```bash
$ cd /home/software/
$ wget http://archive.cloudera.com/cdh4/cdh/4/hadoop-2.0.0-cdh4.6.0.tar.gz
$ sudo mkdir /usr/local/hadoop/
$ tar xzf hadoop-2.0.0-cdh4.6.0.tar.gz 
$ sudo mv hadoop-2.0.0-cdh4.6.0 /usr/local/
$ sudo mv /usr/local/hadoop-2.0.0-cdh4.6.0 /usr/local/hadoop
$ sudo chown -R hadoop:hadoop /usr/local/hadoop
```
创建存储临时文件temp、data和name节点数据的目录

```bash
$ sudo mkdir /usr/local/hadoop/temp/ /usr/local/hadoop/data/ /usr/local/hadoop/name/ 
$ sudo chown -R hadoop:hadoop /usr/local/hadoop
```

好了，准备工作终了

开始配置
配置/usr/local/hadoop/etc/hadoop/core-site.xml

```xml
<configuration>
        <property>
                <name>fs.defaultFS</name>
                <value>hdfs://mdn3namenode1.net:9000</value> <!-- master域名或者master的ip -->
        </property>
        <property>
                <name>io.file.buffer.size</name>
                <value>131072</value>
        </property>
        <property>
                <name>hadoop.tmp.dir</name>
                <value>file:/usr/local/hadoop/temp</value>
                <description>Abase for other temporary directories.</description>
        </property>
        <property>
                <name>hadoop.proxyuser.hduser.hosts</name>
                <value>*</value>
        </property>
        <property>
                <name>hadoop.proxyuser.hduser.groups</name>
                <value>*</value>
        </property>
</configuration>
```

配置/usr/local/hadoop/etc/hadoop/hdfs-site.xml

```xml
<configuration>
        <property>
                <name>dfs.namenode.secondary.http-address</name>
                <value>mdn3namenode1.net:9001</value> <!-- master域名或者master的ip -->
        </property>
        <property>
                <name>dfs.namenode.name.dir</name>
        <value>file:/usr/local/hadoop/dfs/name</value>
        </property>
        <property>
                <name>dfs.datanode.data.dir</name>
                <value>file:/usr/local/hadoop/dfs/data</value>
        </property>
        <property>
                <name>dfs.replication</name>
                <value>3</value>
        </property>
        <property>
                <name>dfs.webhdfs.enabled</name>
                <value>true</value>
        </property>
</configuration>
```

配置/usr/local/hadoop/etc/hadoop/madpred-site.xml

```bash
cp mapred-site.xml.template mapred-site.xml
```

```xml
<configuration>
        <property>
                <name>mapreduce.framework.name</name>
                <value>yarn</value>
        </property>
        <property>
                <name>mapreduce.jobhistory.address</name>
                <value>mdn3namenode1.net:10020</value> <!-- master域名或者master的ip -->
        </property>
        <property>
                <name>mapreduce.jobhistory.webapp.address</name>
                <value>mdn3namenode1.net:19888</value> <!-- master域名或者master的ip -->
        </property>
</configuration>
```

配置/usr/local/hadoop/etc/hadoop/yarn-site.xml

```xml
<configuration>
        <property>
                <name>yarn.nodemanager.aux-services</name>
                <value>mapreduce.shuffle</value>
        </property>
        <property>
                <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
                <value>org.apache.hadoop.mapred.ShuffleHandler</value>
        </property>
        <property>
                <name>yarn.resourcemanager.address</name>
                <value>mdn3namenode1.net:8032</value> <!-- master域名或者master的ip -->
        </property>
        <property>
                <name>yarn.resourcemanager.scheduler.address</name>
                <value>mdn3namenode1.net:8030</value> <!-- master域名或者master的ip -->
        </property>
        <property>
                <name>yarn.resourcemanager.resource-tracker.address</name>
                <value>mdn3namenode1.net:8031</value> <!-- master域名或者master的ip -->
        </property>
        <property>
                <name>yarn.resourcemanager.admin.address</name>
                <value>mdn3namenode1.net:8033</value> <!-- master域名或者master的ip -->
        </property>
        <property>
                <name>yarn.resourcemanager.webapp.address</name>
                <value>mdn3namenode1.net:8088</value> <!-- master域名或者master的ip -->
        </property>
</configuration>
```

编辑slave的名字
直接讲slave的域名或者slave的ip按照一行一个的规则写进去

```bash
mdn3datanode2.net
mdn3datanode3.net
```

复制到各台机器上

```bash
$ cd /usr/local/
$ sudo scp -dr hadoop@192.168.216.183:/usr/local/hadoop .
$ sudo chown -R hadoop:hadoop hadoop/
```

格式化hdfs
在namenode上执行

```bash
/usr/local/hadoop/bin/hadoop namenode -format
```

###hbase的安装配置
hbase依赖zookeeper，需要先去下载

```bash
$ cd /home/software/
$ wget http://archive.cloudera.com/cdh4/cdh/4/zookeeper-3.4.5-cdh4.6.0.tar.gz
$ tar xzf zookeeper-3.4.5-cdh4.6.0.tar.gz
$ sudo mv zookeeper-3.4.5-cdh4.6.0 /usr/local/
$ sudo mv /usr/local/zookeeper-3.4.5-cdh4.6.0 /usr/local/zookeeper
$ sudo chown -R hadoop:hadoop /usr/local/zookeeper
$ sudo cp /usr/local/zookeeper/conf/zoo_sample.cfg /usr/local/zookeeper/conf/zoo.cfg
```
zookeeper准备完毕，可以继续安装hbase

```bash
$ cd /home/software/
$ wget http://archive.cloudera.com/cdh4/cdh/4/hbase-0.94.15-cdh4.6.0.tar.gz
$ sudo mkdir /usr/local/hbase/
$ tar xzf hbase-0.94.15-cdh4.6.0.tar.gz
$ sudo mv hbase-0.94.15-cdh4.6.0 /usr/local/
$ sudo mv /usr/local/hbase-0.94.15-cdh4.6.0 /usr/local/hbase
$ sudo chown -R hadoop:hadoop /usr/local/hbase
```


若干配置步骤
配置hbase-site.xml

```xml
<configuration>
    <property>
        <name>hbase.rootdir</name>
        <value>hdfs://mdn3namenode1.net:9000/hbase</value>

    </property>
    <property>
        <name>hbase.cluster.distributed</name>
        <value>true</value>
    </property>
    <property>
        <name>hbase.master</name>
        <value>mdn3datanode1.net:60000</value>
    </property>
    <property>
        <name>hbase.zookeeper.quorum</name>
        <value>mdn3datanode1.net</value>    <!-- 这里配置若干个zookeeper的服务器地址，需要是奇数个 -->
    </property>
</configuration>
```

配置hbase-env.sh

```bash
export HBASE_MANAGES_ZK=false
```

不要hbase托管zookeeper

配置regionservers

```bash
mdn3datanode2.net
mdn3datanode3.net
```

启动hbase

```bash
/usr/local/hbase/bin/start-hbase.sh
/usr/local/hbase/bin/hbase-daemons.sh start thrift
```
hbase启动完成.


###配置hbase可能碰到几个问题的说明：
1) 报错
` ERROR client.HConnectionManager$HConnectionImplementation: Check the value configured in 'zookeeper.znode.parent' `

是需要把/etc/hosts中的127.0.0.1注释掉，否则zookeeper还会出现
最后的hosts我这里是这样

```bash
[hadoop@localhost conf]$ more /etc/hosts
#127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.216.183 mdn3namenode1.net mdn3datanode1.net
192.168.216.184 mdn3namenode2.net mdn3datanode2.net
192.168.216.185 mdn3datanode3.net mdn3nfsserver.net
```

2) 在运行/usr/local/hbase/bin/hbase shell的时候出现了
` WARN conf.Configuration: hadoop.native.lib is deprecated. Instead, use io.native.lib.available `

3) ` java.net.ConnectException: Connection refused `
这是要求hadoop中的slaves配置和hbase的regionservers要一致。

###hive的安装

```bash
cd /home/software
wget http://archive.cloudera.com/cdh4/cdh/4/hive-0.10.0-cdh4.6.0.tar.gz
tar xzf hive-0.10.0-cdh4.6.0.tar.gz
sudo mv hive-0.10.0-cdh4.6.0 /usr/local/
sudo mv /usr/local/hive-0.10.0-cdh4.6.0 /usr/local/hive
chown -R hadoop:hadoop /usr/local/hive
```

###hive的配置
在~/.bashrc中加入

```bash
export HIVE_HOME=/usr/local/hive
export HIVE_CONF_DIR=$HIVE_HOME/conf
export HIVE_LIB=$HIVE_HOME/lib
export PATH=$PATH:$JAVA_HOME/bin:$ZOOKEEPER_HOME:$HIVE_HOME
```

在conf/hive-site.xml中

```xml
<configuration>
<property>
  <name>hive.metastore.local</name>
  <value>true</value>
</property>

<property>
  <name>javax.jdo.option.ConnectionURL</name>
  <value>jdbc:mysql://localhost:3306/hive</value>
</property>

<property>
  <name>javax.jdo.option.ConnectionDriverName</name>
  <value>com.mysql.jdbc.Driver</value>
</property>

<property>
  <name>javax.jdo.option.ConnectionUserName</name>
  <value>hive</value>
</property>

<property>
  <name>javax.jdo.option.ConnectionPassword</name>
  <value>hive</value>
</property>
<property>
  <name>datanucleus.fixedDatastore</name>
  <value>false</value>
</property>

</configuration>
```

这里要安装mysql作为元数据服务器，参考这篇 http://evoupsight.com/blog/2014/02/17/hadoop0-dot-20-dot-2-plus-hive0-dot-7/

然后/bin/hive后，成功进入shell

```bash
> create table test (key string);
```

如果遇到下面的报错
` FAILED: Error in metadata: java.lang.RuntimeException: Unable to instantiate org.apache.hadoop.hive.metastore.HiveMetaStoreClient `

` FAILED: Execution Error, return code 1 from org.apache.hadoop.hive.ql.exec.DDLTask `
建表错误
开始以为hive没有访问mysql的权限,以root用户登录mysql然后赋予hive用户权限：

```sql
grant all privileges on *.* to hive@localhost identified by 'hive';
grant all privileges on *.* to hive@192.168.216.183 identified by 'hive';
```

发现问题依旧

其实是要在hive-site.xml中把

```xml
<property>
  <name>javax.jdo.option.ConnectionURL</name>
  <value>jdbc:mysql://localhost:3306/hive</value>
</property>
```

改成

```xml
<property>
  <name>javax.jdo.option.ConnectionURL</name>
  <value>jdbc:mysql://192.168.216.183:3306/hive</value>
</property>
```

问题依旧，打开hive的调试模式

```bash
bin/hive -hiveconf hive.root.logger=DEBUG,console
```

` 14/05/08 17:35:53 WARN conf.HiveConf: DEPRECATED: Configuration property hive.metastore.local no longer has any effect. `
` Make sure to provide a valid value for hive.metastore.uris if you are connecting to a remote metastore `

在配置文件里删除hive.metastore.local属性。

最后查得原因是没有安装mysql驱动，只要把mysql-connector-java-5.1.22-bin.jar放到lib下就可以了

然后

```bash
hive> create table test (key string);
OK
Time taken: 42.259 seconds

hive> show tables;
OK
test
Time taken: 0.279 seconds
```

