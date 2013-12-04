---
layout: post
title: "centos6.4 install cacit0.8"
date: 2013-12-04 10:37
comments: true
categories: 
---
还是一样先把不要的关了
```sh
service iptables stop
chkconfig iptables off
vim /etc/selinux/config
```
然后SELINUX=enforcing改成SELINUX=disabled
重启机器getenforce 为disable
正式开篇
```sh
yum -y install mysql mysql-server mysql-devel
yum -y install httpd
yum -y install php php-mysql
```

启动服务并加入到启动列表
```sh
service httpd start
service mysqld start
chkconfig httpd on
chkconfig mysqld on
```

安装需要的库文件
```sh
yum -y install zlib
yum -y install freetype
yum -y install libjpeg
yum -y install fontconfig
yum -y install gd
yum -y install libxml2
yum -y install php-gd
```

安装rrdtool
```sh
yum -y install rrdtool
```

安装snmp支持工具
```sh
yum -y install net-snmp
yum -y install net-snmp-utils
```

启动snmpd服务
```sh
service snmpd start
chkconfig snmpd on
```

安装cacti
下载wget http://www.cacti.net/downloads/cacti-0.8.7e.tar.gz

```sh
tar xzf cacti-0.8.7e.tar.gz
mv cacti-0.8.7e /var/www/html/cacti
cd /var/www/html
vim cacti/include/config.php
```
编辑为下面的配置
```php
$database_type = "mysql";
$database_default = "cacti";
$database_hostname = "localhost";
$database_username = "cacti";
$database_password = "cacti";
$database_port = "3306";
```

添加cacti用户
useradd cacti

添加计划任务
编辑/etc/crontab
```sh
* * * * * cacti /usr/bin/php /var/www/html/cacti/poller.php >> /dev/null 2>&1
```

更改属组
将web目录的所属组改为cacti
```sh
#chgrp -R cacti /var/www/html/cacti
#chown -R cacti /var/www/html/cacti/rra
#chown -R cacti /var/www/html/cacti/log
#chown -R cacti /var/www/html/cacti/poller.php
```

创建cacti数据库
```sh
mysql
mysql> create database cacti;
```
导入cacti.sql
```sh
mysql -uroot -p cacti < /var/www/html/cacti/cacti.sql
mysql> use cacti
mysql> grant all on cacti.* to cacti@localhost identified by 'cacti';
mysql> flush privileges;
mysql> exit
```

浏览器登录cacti
http://ip/cacti
new install后一路回车,安装完成。
默认登录密码为admin/admin

###疑难杂症之cacti不出图
首先检查log文件权限
```sh
cd /var/www/html/cacti/log
ll
-rw-r--r-- 1 cacti cacti 438 Dec  3 02:14 cacti.log
```

校正时间
```sh
#cp -f /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
#reboot
```


确保/var/www/html/cacti/rra目录下有可写权限。
重启点击graph菜单观察发现已经出现图表了。


