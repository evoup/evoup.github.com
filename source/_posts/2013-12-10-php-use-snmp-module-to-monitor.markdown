---
layout: post
title: "使用php的snmp模块进行监控数据的获取"
date: 2013-12-10 10:19
comments: true
categories: [php,monitor] 
---
简单地根据经验归纳，snmp被叫做简单网络管理协议(Simple Network Management Protocol)，是监控业界标准的设备间通讯接口会话层协议，位于OSI的第五层。各类开源软件广泛采用了此协议进行数据的获取，目前已经从明文传输的v1发展到了具有较高安全性的v3版本。与snmp相关的知识有很多，` MIB `（管理信息数据库），` OID `（对象标识符），就目前而言，只需要记得有这2个名字就可以了。
通过源码安装php的snmp模块和安装一般php的模块没有本质区别。一共也就2种方式，--enable或者--with静态方式和phpize动态方式。共同点是都需要ext目录下面有snmp这个模块。
下面先论述如以--enable或者--with的方式来把snmp静态编译到php中去。
首先下载php软件包。
<!-- more -->
```sh
wget wget http://tw1.php.net/get/php-5.5.6.tar.bz2/from/this/mirror
tar xjf php-5.5.6.tar.bz2
cd php-5.5.6
$ ls ext/snmp/
config.m4  config.w32  CREDITS  php_snmp.h  snmp.c  snmp.dsp  tests
$
```
可见snmp已经自带了，不需要到pecl下载然后放到ext目录。
```sh
sudo yum install libxml2 libxml2-devel 
```
备注如果一意孤行，只装libxml2，呵呵，那么你将收到` configure: error: xml2-config not found. Please check your libxml2 installation. `的报错。无独有偶，你要是不装net-snmp-devel，就可以收到` configure: error: Could not find net-snmp-config binary. Please check your net-snmp installation `的报错。 

###静态编译
查一下snmp的安装选项
```sh
$ ./configure --help | grep snmp
  --with-snmp=DIR         Include SNMP support
```
是--with，了解后继续操作，注意要指定好snmp路径
```sh
./configure --prefix=/usr/local/php55_static_snmp --with-snmp --ebable-sockets 
make
sudo make install
```
这样静态编译就完成了。

###动态编译
假设一开始把php安装在/usr/local/php55，现在要以编译出sockets.so和snmp.so
```sh
cd ext/snmp
/usr/local/php55/bin/phpize
./configure --with-php-config=/usr/local/php55/bin/php-config
make
sudo make install
cd ../../ext/sockets
/usr/local/php55/bin/phpize
./configure --with-php-config=/usr/local/php55/bin/php-config
make
sudo make install

```
然后复制刚才的编译好的so文件到扩展目录到etc目录
```sh
cp /usr/local/php55/lib/php/extensions/no-debug-non-zts-20121212/snmp.so /usr/local/php55/etc/
cp /usr/local/php55/lib/php/extensions/no-debug-non-zts-20121212/sockets.so /usr/local/php55/etc/
```
编辑php的配置文件，加入2行
```sh
vi /usr/local/php55/etc/php.ini
```
加入
```sh
extension=sockets.so 
extension=snmp.so
```

###撰写php版的snmp客户端测试程序
开始写点测试程序，确认snmp已经在本机支持
```sh
$nc -uvz 127.0.0.1 161
Connection to 127.0.0.1 161 port [udp/snmp] succeeded!
```
已经支持，那么来写程序吧
```php
<?php
$host="127.0.0.1";
$community="public";
$oid=".1.3.6.1.4.1.2021.10.1.3.1";
$oid1=".1.3.6.1.4.1.2021.10.1.3.2";
$oid2=".1.3.6.1.4.1.2021.10.1.3.3";
$oid3=".1.3.6.1.4.1.2021.4.3.0";
echo (snmpget($host,$community,$oid)."\n");  //1 minute Load
echo (snmpget($host,$community,$oid1)."\n"); //5 minute Load
echo (snmpget($host,$community,$oid2)."\n"); //15 minute Load
echo (snmpget($host,$community,$oid3)."\n"); //Total Swap Size
```
查看结果
```sh
$/usr/local/php55/bin/php test.php
STRING: 0.05
STRING: 0.03
STRING: 0.03
INTEGER: 2064376 kB


```
竟然是如此简单，由此推断你只要学会rrdtool和php snmp，自行打造一款类cacti的监控软件不会有太大的困难。但是这么做其实还有一点要注意，得装snmpd，我们自己做公司监控平台2.0的时，上级要求使用C api直接获取数据，不走snmp，其实分析ganglia其实也是采用了类似的方式，所以号称比snmpd快，为了完成任务和系统的拓展性，果然支持snmp吧。分析完毕。

扩展阅读：请需要OID对照资料的兄弟自行互联网查询《linux常用OID》

