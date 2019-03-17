---
layout: post
title: "php和zookeeper交互获取hbase的master状态"
date: 2013-03-25 10:54
comments: true
categories: [php,hadoop,zookeeper] 
---

###为什么要使用zookeeper扩展
公司用了HHVM这玩意来编译php，其实我要给它NO，因为连zookeeper都没支持，如果要支持，要自己写扩展，这是一个奇葩的节奏。所以我们还是采用php脚本方式来调用pecl扩展来实现php和zookeeper通讯。
<!-- more -->

###扩展
以下是php的pecl版zookeeper扩展的下载地址http://pecl.php.net/get/zookeeper-0.2.2.tgz<br>
可以看到php版本的要求是>5.2.0

```bash
cd /home/software
wget http://pecl.php.net/get/zookeeper-0.2.2.tgz
tar xzf zookeeper-0.2.2.tgz
```

###依赖库
很可惜，该扩展的安装还需要你先去在本地下好zookeeper依赖库，那么我们开始吧。首先是zookeeper的安装，去apache下载好并解压<br>

```bash
cd /home/software
wget http://apache.fayea.com/apache-mirror/zookeeper/zookeeper-3.4.6/zookeeper-3.4.6.tar.gz
tar xzf zookeeper-3.4.6.tar.gz
cd zookeeper-3.4.6/src/c
./configure --prefix=/home/software/zookeeper-3.4.6/prefix
sudo make install
```

###php端编译
这里就给一个静态编译的例子好了,以最新版本的php5.3.8为例，进入源码文件夹后

```bash
$ cd /home/software/php-5.3.8/
$ cp -r /home/software/zookeeper-0.2.2 ext/zookeeper
$ ls ext/zookeeper/
CREDITS                 LICENSE                 config.m4               php_zookeeper.c         php_zookeeper_private.h php_zookeeper_session.h zoo_lock.h
ChangeLog               README.markdown         examples                php_zookeeper.h         php_zookeeper_session.c zoo_lock.c              zookeeper-api.php
$ ./buildconf -force
$ './configure'  '--prefix=/usr/local/php5.3.8_zookeeper' '--enable-zookeeper' '--with-libzookeeper-dir=/home/software/zookeeper-3.4.3/prefix' '--enable-sockets'
$ make
$ sudo make install
```

安装完成后查看是否支持

```bash
$ /usr/local/php5.3.8._zookeeper/bin/php -i | grep 'libzookeeper version'
libzookeeper version => 3.4.3
```
看到已经支持了

###最终的获取
找一台非托管zk的hbase，这里假设是127.0.0.1，端口为2181

{% codeblock lang:php %}
<?php
class zookeeper_instance extends Zookeeper {
    function connect_cb($type, $event, $string) {
        if ($event == Zookeeper::CONNECTED_STATE) {
            $acl=array(
                "perms"=>0x1f,
                "scheme"=>"world",
                "id"=>"anyone"
            );
        }
    }
}

$zk=new zookeeper_instance();
echo "instance ok\n";

$zk->connect("127.0.0.1:2181", array($zk, 'connect_cb'),200000); //连接超时200秒,比较夸张，测试用：）
echo "connect ok\n";
$zkm=$zk->get("/hbase/master");
print_r($zkm);
?>
{% endcodeblock %}

查看结果,已经获取到了master

```bash
[yin@yin-arch php_zookeeper_sample]>/usr/home/yin/local/bin/php5_new/bin/php test_zk_gethbasemaster.php
instance ok
connect ok
▒25469@namenode1namenode1,60000,1395387861310[yin@yin-arch php_zookeeper_sample]>
```

收工!
