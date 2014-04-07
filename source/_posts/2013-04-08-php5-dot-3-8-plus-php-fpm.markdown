---
layout: post
title: "Php5.3.8+php-fpm源码安装"
date: 2013-04-08 01:03
comments: true
categories: php
---
 
本文在freebsd9.2 i386下进行。选择php5.3.8才能正常编译

### pecl扩展(以下的pecl源码改名后放到php源码的ext目录下)
i.memcache-3.0.4->memcache<br>
ii.APC-3.1.4->apc<br>
iii.imagick-3.0.0->imagick<br>
(已经把memcache,apc模块源码放到ext子目录中,名称如上)

```sh
$ cd /your/phpsrcdir
$ rm configure
$ ./buildconf --force
$ ./configure --help   #(选项出现了!)
```

### 预装软件：
ImageMagick
```sh
$ make WITHOUT_X11=yes install clean
```

(时间较长，干点别的去)
libevent(php-fpm需要,ports)
```sh
$ make install clean
```

curl(ports)
```sh
$ make install clean
```

libmcrypt(ports)
```sh
make install clean
```

● 手动编译php,每个参数都要知道具体是干嘛的,5.3.3之后，有了php-fpm，
--enable-fastcgi就不再需要了，mysqli以及pdo-mysql都用mysqlnd
```sh
$./configure' '--prefix=/usr/local/php5_admin' '--with-layout=GNU' 
'--with-config-file-scan-dir=/usr/local/php5_admin/etc/php' '--disable-all' '--enable-dom'  
'--enable-filter' '--enable-hash' '--enable-json' '--with-mcrypt' '--with-curl'  
'--with-pcre-regex' '--enable-mbstring' '--enable-ctype' '--enable-session' 
'--enable-libxml' '--enable-simplexml' '--enable-pdo' '--with-pdo-mysql=mysqlnd' 
'--with-mysqli=mysqlnd' '--enable-sysvsem' '--enable-sysvshm' '--enable-apc' 
'--enable-memcache' '--with-imagick=/usr/local' '--enable-fpm'
```

注意
make时候会报错
解决方法：
修改fpm_sockets.c代码：
1 . i n f o . t c p i _ s a c k e d     = >   i n f o . _ _ t c p i _ s a c k e d  
2 . i n f o . t c p i _ u n a c k e d   = >   i n f o . _ _ t c p i _ u n a c k e d  
● 配置
● #cp /usr/local/php5_admin/etc/php-fpm.conf.default 
/usr/local/php5_admin/etc/php-fpm.conf
  ●
● 几个需要配置的参数
● pm.max_children
● pm.start_servers
● pm.min_spare_servers
● pm.max_spare_servers
启动脚本
/usr/local/etc/rc.d/phpfpm
#!/bin/sh
# PROVIDE: phpfpm
# REQUIRE: DAEMON
#
# Add the following lines to /etc/rc.conf to run phpfpm:
#
# phpfpm_enable (bool):   Set it to "YES" to enable {phpfpm}.
#               Default is "NO".
#
# Last-Modified: 2010-09-14 23:30:20
name="phpfpm" 
. /etc/rc.subr
rcvar=`set_rcvar`
load_rc_config ${name}
eval ${name}_enable=\${${name}_enable:-"NO"}
eval server=\${${name}_server:-"/usr/local/php5_admin/sbin/php-fpm"}
command=${server}
extra_commands="reload" 
sig_reload="USR2" 
pidfile="/usr/local/php5_admin/var/run/php-fpm.pid" 
#command_args="" 
run_rc_command "$1" 
   注意：启动脚本的权限555，还有/usr/local/php5_admin/var/run/php-fpm.pid可能需要手动
创建
在/etc/rc.conf中加入phpfpm_enable="YES"
#/usr/local/etc/rc.d/phpfpm start
启动ok
备注，其实：上面的phpfpm配置文件是错误的，原因如下：
php 5.3.3以上的php版本源码中已经内嵌了 php-fpm，不用象以前的php版本一样专门打补丁了，只需要在
configure的时候添加编译参数即可。
但是，php 5.3.3以上版本的php-fpm 不再支持 php-fpm 以前具有的 /usr/local/php/sbin/php-fpm 
(start|stop|reload)等命令，需要使用信号控制：
master进程可以理解以下信号：
● INT, TERM 立刻终止
● QUIT 平滑终止
● USR1 重新打开日志文件
● USR2 平滑重载所有worker进程并重新载入配置和二进制模块
示例：
注意这边的单引号为“esc下面那个键（~）”
php-fpm 关闭：
k i l l   - I N T   ` c a t   / u s r / l o c a l / p h p / v a r / r u n / p h p - f p m . p i d `
php-fpm 重启：
k i l l   - U S R 2   ` c a t   / u s r / l o c a l / p h p / v a r / r u n / p h p - f p m . p i d `
查看php-fpm进程：
p s   a u x   |   g r e p   - c   p h p - f p m
查看php-fpm进程数：
p s   a u x   |   g r e p   - c   p h p - f p m   |   w c   - l
正确的脚本应该是这样的
#!/bin/sh
# PROVIDE: phpfpm
# REQUIRE: DAEMON
#
# Add the following lines to /etc/rc.conf to run phpfpm:
#
# phpfpm_enable (bool):   Set it to "YES" to enable {phpfpm}.
#               Default is "NO".
#
# Last-Modified: 2012-11-22 10:27:43
name="phpfpm_monitor_client"
. /etc/rc.subr
rcvar=`set_rcvar`
load_rc_config ${name}
eval ${name}_enable=\${${name}_enable:-"NO"}
eval server=\${${name}_server:-"/usr/local/php5_admin/sbin/php-fpm"}
command=${server}
extra_commands="reload"
sig_reload="USR2"
sig_stop="INT"
pidfile="/usr/local/php5_admin/var/run/php-fpm-monitor-client.pid"
fpmconffile="/usr/local/php5_admin/etc/php-fpm_monitor_client.conf"
command_args="-g ${pidfile} -y ${fpmconffile}"
#run_rc_command $1
run_rc_command $*