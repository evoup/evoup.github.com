---
layout: post
title: "centos6.4下搭建zabbix2.0.6"
date: 2013-12-03 13:37
comments: true
categories: monitor 
---
最近开始研究各款监控系统，和自己的监控系统进行横向比较找差距，就从单位也在用的zabbix开始，先安装后再这里过一遍流程。

<!-- more -->
关闭iptables和selinux

```sh
sudo service iptables stop
sudo chkconfig iptables off
```

临时关闭selinux

```sh
setenfoce 0  ##设置为permissive模式
             ##setenforce 1设置SELunix为enforcing模式
```

永久关闭
修改/etc/selinux/config 文件
将SELINUX=enforcing改为SELINUX=disabled
重启机器看到
getenforce返回Disabled说明已经关闭

再次进入centos6.4，先安装LAMP环境。

```sh
yum install -y httpd mysql mysql-server mysql-devel php php-mysql php-common php-mbstring php-gd php-odbc php-xml php-pear
```

然后下载zabbix

```sh
wget http://sourceforge.net/projects/zabbix/files/ZABBIX%20Latest%20Stable/2.0.6/zabbix-2.0.6.tar.gz/download
```

继续安装需要的组件

```sh
yum install -y curl curl-devel net-snmp net-snmp-devel perl-DBI
```

创建zabbix用户帐号

```sh
sudo useradd zabbix
sudo usermod -s /sbin/nologin zabbix
```

启动mysql

```sh
sudo service mysqld start
```

登录mysql

```sh
mysql -uroot -p123456
mysql> create database zabbix;
mysql> grant all on zabbix.* to zabbix@localhost identified by '123456';
mysql> use zabbix;
mysql> source /home/evoup/Downloads/zabbix-2.0.6/database/mysql/schema.sql
mysql> source /home/evoup/Downloads/zabbix-2.0.6/database/mysql/images.sql
mysql> source /home/evoup/Downloads/zabbix-2.0.6/database/mysql/data.sql
mysql> exit
```

安装zabbix

```sh
./configure --enable-server --enable-agent --with-mysql --with-net-snmp --with-libcurl
Configuration:

  Detected OS:           linux-gnu
  Install path:          /usr/local
  Compilation arch:      linux

  Compiler:              gcc
  Compiler flags:        -g -O2  -I/usr/include/mysql  -g -pipe -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector --param=ssp-buffer-size=4 -m64 -D_GNU_SOURCE -D_FILE_OFFSET_BITS=64 -D_LARGEFILE_SOURCE -fno-strict-aliasing -fwrapv -fPIC   -DUNIV_LINUX -DUNIV_LINUX       -I/usr/include/rpm -I/usr/local/include -I/usr/lib64/perl5/CORE -I. -I/usr/include  

  Enable server:         yes
  Server details:
    With database:         MySQL
    WEB Monitoring via:    cURL
    Native Jabber:         no
    SNMP:                  net-snmp
    IPMI:                  no
    SSH:                   no
    ODBC:                  no
    Linker flags:          -rdynamic      -L/usr/lib64/mysql       -L/usr/lib64  -L/usr/lib64
    Libraries:             -lm -lrt  -lresolv    -lmysqlclient       -lcurl  -lnetsnmp -lcrypto  -lnetsnmp -lcrypto

  Enable proxy:          no

  Enable agent:          yes
  Agent details:
    Linker flags:          -rdynamic
    Libraries:             -lm -lrt  -lresolv    -lcurl

  Enable Java gateway:   no

  LDAP support:          no
  IPv6 support:          no

***********************************************************
*            Now run 'make install'                       *
*                                                         *
*            Thank you for using Zabbix!                  *
*              <http://www.zabbix.com>                    *
***********************************************************
```

好了，我们正式安装

```sh
sudo make install
```

接下来编辑配置文件
可以不用sudo，直接切换到root做

```sh
cd /usr/local/etc
cat zabbix_server.conf
LogFile=/var/log/zabbix_server.log
DBHost=localhost
DBName=zabbix
DBUser=zabbix
DBPassword=123456

cat zabbix_agent.conf
LogFile=/tmp/zabbix_agentd.log
Server=127.0.0.1
UnsafeUserParameters=1
```

创建日志文件

```sh
touch /var/log/zabbix_server.log
touch /var/log/zabbix_agent.log

cd -
cp misc/init.d/tru64/zabbix_server /etc/init.d/
cp misc/init.d/tru64/zabbix_agentd /etc/init.d/
```

2个文件的文件头改成如下

```sh
#!/bin/sh
#chkconfig: 35 95 95
#description:zabbix Agent server
```

添加服务

```sh
chkconfig --add zabbix_server
chkconfig --add zabbix_agentd
```

开机自动启动

```sh
chkconfig zabbix_server on
chkconfig zabbix_agent on
```

文件执行权限

```sh
chmod +x zabbix_server
chmod +x zabbix_agentd
```

启动

```sh
sudo /etc/init.d/zabbix_server start
sudo /etc/init.d/zabbix_agentd start
```

安装zabbix web

```sh
cp -r frontends/php/ /var/www/html/zabbix
```

访问http://ip/zabbix

```sh
vim /etc/php.ini
```
指定

```php
timezone = Asia/Shanghai
```

根据提示修改php配置

```sh
yum install -y php-bcmath
```

然后再次重启httpd

最后登录前端界面密码为 admin/zabbix

告一段落，接下来学习操作zabbix。


