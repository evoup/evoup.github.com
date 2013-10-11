---
layout: post
title: "freebsd install ganglia3.4"
date: 2012-10-22 11:24
comments: true
categories:    monitor
---
##Ganglia是什么?
简单的说，这是一个开源的系统监控软件，本身通过rrdtool这个软件作为数据载体，以及SNMP协议采集监控数据，最终在管理界面上呈现出监控图表数据的系统。

<!-- more -->

##安装小记
首先是下载ganglia
{% codeblock shell lang:bash %}
wget http://downloads.sourceforge.net/project/ganglia/
wget http://oss.oetiker.ch/rrdtool/pub/rrdtool-1.4.7.tar.gz
{% endcodeblock %}

先安装rrdtool
{% codeblock shell lang:bash %}
./configure --prefix=/usr/lib32/rrdtool --disable-perl
make && make install
{% endcodeblock %}

(Ps:安装过程有缺少包的解决，可以尝试先从port安装好rrdtool之后deinstall rrdtool，这样依赖的包就都有了，因为port直接安装rrdtool后ganglia不认，所以手动编译)

再来是ganglia
{% codeblock shell lang:bash %}
./configure --with-gmetad --with-librrd=/usr/lib32/rrdtool
make && make install
{% endcodeblock %}
报告

Checking for apr

checking for apr-1-config... no

configure: error: apr-1-config binary not found in path

没有安装apache2

先安装mysql

whereis mysql51-server
{% codeblock shell lang:bash %}
wget ftp://ftp.fi.muni.cz/pub/mysql/Downloads/MySQL-5.1/mysql-5.1.60.tar.gz
sudo make WITH_CHARSET=utf8 WITH_XCHARSET=all install clean
sudo make install
{% endcodeblock %}

安装httpd
{% codeblock shell lang:bash %}
wget http://labs.renren.com/apache-mirror/httpd/httpd-2.2.23.tar.gz
./configure --prefix=/usr/local/apache2 --enable-modules=so --enable-rewrite
make && make install
{% endcodeblock %}

安装php
{% codeblock shell lang:bash %}
wget http://cn.php.net/get/php-5.3.6.tar.gz/from/this/mirror
tar –zxvf php-5.3.6.tar.gz
cd php-5.3.6
./configure --prefix=/usr/local/php --with-apxs2=/usr/local/apache2/bin/apxs --with-config-file-path=/usr/local/lib --with-mysql=/usr
make && make install
{% endcodeblock %}

将php5的库复制到apache的modules里
{% codeblock shell lang:bash %}
sudo cp –p .libs/libphp5.so /usr/local/apache2/modules
sudo chown cdh:cdh /usr/local/apache2/modules/libphp5.so
{% endcodeblock %}

复制php5配置文件
{% codeblock shell lang:bash %}
sudo cp php.ini-development /usr/local/php/lib/php/php.ini
{% endcodeblock %}

修改http.conf 兼容php5
{% codeblock shell lang:bash %}
sudo vim /usr/local/apache2/conf/httpd.conf 
{% endcodeblock %}
加上
{% codeblock shell lang:bash %}
AddType application/x-httpd-php .php  
#LoadModule php5_module modules/libphp5.so
{% endcodeblock %}  
把上面的#号去掉  

 

DirectoryIndex index.html 

在后面加 index.php

 

DocumentRoot "/usr/local/apache2/htdocs"  

把/usr/local/apache2/htdocs改为你存放网页文件的路径  

 

AddDefaultCharset iso8859-1  

把后面的iso8859-1改为gb2312 或者是干脆off 

更详细的请参考http://article.21e.cn

 

到这里看下访问一下浏览器，php应该工作ok

 

继续编译ganglia
{% codeblock shell lang:bash %}
sudo ./configure --with-gmetad --with-librrd=/usr/lib32/rrdtool
{% endcodeblock %} 
configure: error: apr-1-config binary not found in path

{% codeblock shell lang:bash %}
whereis apr1
cd /usr/ports/devel/apr1
sudo make install clean && rehash
{% endcodeblock %} 
 

回来继续编译，报告缺少libconfuse not found
{% codeblock shell lang:bash %}
cd /usr/ports/devel/libconfuse/
sudo make install clean && rehash
{% endcodeblock %} 
 

报告缺少expat.h
{% codeblock shell lang:bash %}
cd /usr/ports/textproc/expat2
sudo make install clean && rehash
{% endcodeblock %} 

问题依旧修改配置
{% codeblock shell lang:bash %}
sudo ./configure --with-gmetad --with-librrd=/usr/lib32/rrdtool --with-libexpat=/usr/local/
sudo make
sudo make install
{% endcodeblock %} 
安装完成！

测试gmetad的运行
{% codeblock shell lang:bash %}
sudo /usr/local/sbin/gmetad -d 5

Going to run as user nobody

Please make sure that /var/db/ganglia/rrds exists: No such file or directory

sudo mkdir -p /var/db/ganglia/rrds

sudo /usr/local/sbin/gmetad -d 5

Going to run as user nobody

Please make sure that /var/db/ganglia/rrds is owned by nobody

sudo chown -R nobody:nobody /var/db/ganglia/

Going to run as user nobody

Sources are ...

Source: [my cluster, step 15] has 1 sources

        127.0.0.1

xml listening on port 8651

interactive xml listening on port 8652

cleanup thread has been started

Data thread 34460440576 is monitoring [my cluster] data source

        127.0.0.1

data_thread() for [my cluster] failed to contact node 127.0.0.1

data_thread() got no answer from any [my cluster] datasource

data_thread() for [my cluster] failed to contact node 127.0.0.1

data_thread() got no answer from any [my cluster] datasource

data_thread() for [my cluster] failed to contact node 127.0.0.1

data_thread() got no answer from any [my cluster] datasource

sudo /usr/local/sbin/gmond -d 5
{% endcodeblock %} 

看上去运行正常了 

基本可以。

开始装界面。不要装3.5.3有些图开不了，3.5.2经过测试没有问题。

http://sourceforge.net/projects/ganglia/files/ganglia-web/3.5.2/ganglia-web-3.5.2.tar.gz/download

 

注意web要求rrd数据库的路径只能是/var/lib/ganglia/rrds

由于之前是装在了/var/db/ganglia/rrds

则
{% codeblock shell lang:bash %}
cd /var/lib
sudo ln -s /var/db/ganglia ganglia
{% endcodeblock %} 

然后访问浏览器

http://192.168.174.133/ganglia/

此时

There was an error collecting ganglia data (127.0.0.1:8652): fsockopen error: Connection refused

 
因为没有打开gmond和gmetad，打开发现界面ok了，但是没有数据！

怀疑是php没有安装rrdtool的扩展,继续安装php的rrdtool扩展

http://pecl.php.net/package/rrd
{% codeblock shell lang:bash %}
sudo ./configure --with-php-config=/usr/local/php/bin/php-config --with-rrd-binary=/usr/lib32/rrdtool/bin/rrdtool
{% endcodeblock %} 

配置报错找不到rrdtool的头文件，直接定位configure文件进行修改

for i in /usr /usr/local /usr/local/rrdtool /opt; do

改成我装的路径

for i in /usr /usr/local /usr/local/rrdtool /opt /usr/lib32/rrdtool; do

改了之后

configure: error: rrd lib version seems older than 1.3.0, update to 1.3.0+

通过后还是报错

error: too many arguments to function 'rrd_lastupdate'

参考这里https://bugs.php.net/bug.php?id=59558

 
```c
- if (rrd_lastupdate(2, &argv[1], &last_update, &ds_cnt, &ds_namv,

+ if (rrd_lastupdate_r(argv[1], &last_update, &ds_cnt, &ds_namv,
```

装完看到phpinfo里有了。

之后还要修改conf_default.php文件

graphite_rrd_dir变量，指向ganglia存放rrd的目录，我的是/var/db/ganglia/rrds

还有一个$conf['rrdtool']变量，指向rrdtool的安装位置，我的是/usr/lib32/rrdtool/bin/rrdtool

应该就差不多了！这样如果还不行，检查自身人品，然后看下/var/log/message