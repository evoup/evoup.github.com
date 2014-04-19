---
layout: post
title: "在操作系统之间迁移二进制程序之一"
date: 2013-04-18 17:52
comments: true
categories: freebsd
---

###写在前面
乍一看又标题党了，但以下文章虽然提到了一个过时的软件pcc2.0，实际我们只是用通过研究程序内部的工作机制，进行举一反三，以再今后的工作中能够起到触类旁通、抛砖引玉的功效。<br>
单位有台机器HostA上安装了鲜为人知的软件叫做pcc2.0，这是一个php的编译器，虽然有许多bug，仍旧采用人工hack的方式使用它。但是目前已经找不到安装包了，怎么办？难道要叫运维帮忙克隆系统吗，有没有办法把程序直接移植到另一台机器HostB上去？答案是肯定的。

<!-- more -->
###程序定位和复制
####主程序的复制
这个程序就是pcc，首先找一下pcc的路径。
```sh
$ whereis pcc
pcc: /usr/bin/pcc /usr/ports/lang/pcc
```

可见pcc所在/usr/bin/目录下，先把这个复制过去
```sh
$ sudo scp user@hostA:/usr/bin/pcc /usr/bin/
```
####程序依赖库的复制
如果这个程序是静态方式编译的，就这样完成了，而如果是动态编译的话，就需要拷贝其他动态库使它能够运行了。怎么确定是否是静态编译还是动态编译？很简单:使用ldd(在HostA上)，我们看一下吧：
```sh
$ ldd /usr/bin/pcc
/usr/bin/pcc:
        libphp-runtime_u.so => /usr/lib/libphp-runtime_u.so (0x28083000)
        libprofiler_u.so => /usr/lib/libprofiler_u.so (0x280ee000)
        libwebconnect_u.so => /usr/lib/libwebconnect_u.so (0x280f2000)
        libphpeval_u.so => /usr/lib/libphpeval_u.so (0x28108000)
        libbigloo_u-2.6f.so => /usr/lib/libbigloo_u-2.6f.so (0x28250000)
        libbigloogc-2.6f.so => /usr/lib/libbigloogc-2.6f.so (0x2833a000)
        libm.so.4 => /lib/libm.so.4 (0x2835c000)
        libc.so.6 => /lib/libc.so.6 (0x28372000)
```
好的，我们登进HostB，可以看到小小一个pcc软件依赖了这么多的so文件，所以这些文件以及这些文件的依赖库都是要复制过去的。值得一提的是，pcc使用了libbigloo库，而它的版本是2.6f，所以我们要先安装好。

```sh
$ cd /home/software
$ fetch https://pkgs.fedoraproject.org/repo/pkgs/bigloo/bigloo2.6f.tar.gz/bc99b1919adee864dd371aeebc36862c/bigloo2.6f.tar.gz
$ tar xzf bigloo2.6f.tar.gz
$ cd bigloo2.6f
$ ./configure --jvm=no
*** ERROR:configure:the C compiler (/usr/local/bin/gcc44) does not seem to work. Aborting
$ whereis gcc
gcc: /usr/bin/gcc /usr/share/man/man1/gcc.1.gz /usr/src/contrib/gcc
$ sudo ln -s /usr/bin/gcc /usr/local/bin/gcc44
$ ./configure --jvm=no
$ gmake
$ sudo gmake install
```

安装完成返回来继续复制，要注意已经存在的不能去盖了
```sh
$ ldd /usr/bin/pcc
/usr/bin/pcc:
        libphp-runtime_u.so => not found (0x0)
        libprofiler_u.so => not found (0x0)
        libwebconnect_u.so => not found (0x0)
        libphpeval_u.so => not found (0x0)
        libbigloo_u-2.6f.so => /usr/local/lib/libbigloo_u-2.6f.so (0x28083000)
        libbigloogc-2.6f.so => /usr/local/lib/libbigloogc-2.6f.so (0x2816d000)
        libm.so.4 => /lib/libm.so.4 (0x2818f000)
        libc.so.6 => /lib/libc.so.6 (0x281a5000)
```

好的，只要not found的我们从HostA复制过来
```sh
$ sudo scp user@hostA:/usr/lib/libphp-runtime_u.so /usr/lib/
$ sudo scp user@hostA:/usr/lib/libprofiler_u.so /usr/lib/
$ sudo scp user@hostA:/usr/lib/libwebconnect_u.so /usr/lib/
$ sudo scp user@hostA:/usr/lib/libphpeval_u.so /usr/lib/
```
然后对于每个新的so，需要再进一步用ldd检查依赖，如果没有的也要复制过来。
试试看，其实已经可以了：
![Alt text](/images/evoup/pcc_result.png)

但是要真正能使用这个软件，还需要复制/opt过来
```sh
scp -dr user@hostA:/opt /
sudo chown root -R /opt
```
然后写一个php测试脚本
```php
<?php
phpinfo();
```
编译
```sh
$ pcc --static index.php -o index
>./index
Runtime error in file index.php on line 2: lookup-function - undefined function: phpinfo
```

看来还有些库没有包含,进入/opt/roadsend/pcc/libs/果然发现很多库，要放在什么地方呢？一般是/usr/lib/
再看一下/opt/roadsend/pcc/bin/roadsend-pcc-uninstall.sh

{% codeblock %}
#
# This is an automatically generated uninstaller script for the
# Roadsend PHP Compiler package
#

## require root
me=`id -u`
if [ "$me" -ne 0 ]
then
    echo "You must be root to run uninstaller"
    exit 1
fi

echo "Uninstalling Roadsend Compiler Runtime..."

rm -f /./opt/roadsend/pcc/libs/libcgi_u.so
rm -f /./opt/roadsend/pcc/libs/libfastcgi_u.so
rm -f /./opt/roadsend/pcc/libs/libmhttpd_u.so
rm -f /./opt/roadsend/pcc/libs/libphp-curl_u.so
rm -f /./opt/roadsend/pcc/libs/libphp-gtk_u.so
rm -f /./opt/roadsend/pcc/libs/libphp-mysql_u.so
rm -f /./opt/roadsend/pcc/libs/libphp-pcre_u.so
rm -f /./opt/roadsend/pcc/libs/libphp-runtime_u.so
rm -f /./opt/roadsend/pcc/libs/libphp-std_u.so
rm -f /./opt/roadsend/pcc/libs/libphp-xml_u.so
rm -f /./opt/roadsend/pcc/libs/libphpeval_u.so
rm -f /./opt/roadsend/pcc/libs/libprofiler_u.so
rm -f /./opt/roadsend/pcc/libs/libwebconnect_u.so
rm -f /./opt/roadsend/pcc/libs/libcgi_u.a
rm -f /./opt/roadsend/pcc/libs/libfastcgi_u.a
rm -f /./opt/roadsend/pcc/libs/libmhttpd_u.a
rm -f /./opt/roadsend/pcc/libs/libphp-curl_u.a
rm -f /./opt/roadsend/pcc/libs/libphp-gtk_u.a
rm -f /./opt/roadsend/pcc/libs/libphp-mysql_u.a
rm -f /./opt/roadsend/pcc/libs/libphp-pcre_u.a
rm -f /./opt/roadsend/pcc/libs/libphp-runtime_u.a
rm -f /./opt/roadsend/pcc/libs/libphp-std_u.a
rm -f /./opt/roadsend/pcc/libs/libphp-xml_u.a
rm -f /./opt/roadsend/pcc/libs/libphpeval_u.a
rm -f /./opt/roadsend/pcc/libs/libprofiler_u.a
rm -f /./opt/roadsend/pcc/libs/libwebconnect_u.a
rm -f /./opt/roadsend/pcc/libs/php-curl.sch
rm -f /./opt/roadsend/pcc/libs/php-gtk.sch
rm -f /./opt/roadsend/pcc/libs/php-mysql.sch
rm -f /./opt/roadsend/pcc/libs/php-pcre.sch
rm -f /./opt/roadsend/pcc/libs/php-runtime.sch
rm -f /./opt/roadsend/pcc/libs/php-std.sch
rm -f /./opt/roadsend/pcc/libs/php-xml.sch
rm -f /./opt/roadsend/pcc/libs/cgi.heap
rm -f /./opt/roadsend/pcc/libs/fastcgi.heap
rm -f /./opt/roadsend/pcc/libs/mhttpd.heap
rm -f /./opt/roadsend/pcc/libs/php-curl.heap
rm -f /./opt/roadsend/pcc/libs/php-gtk.heap
rm -f /./opt/roadsend/pcc/libs/php-mysql.heap
rm -f /./opt/roadsend/pcc/libs/php-pcre.heap
rm -f /./opt/roadsend/pcc/libs/bigloo/2.6f/bigloo.h
rm -f /./opt/roadsend/pcc/libs/bigloo/2.6f/bigloo_config.h
rm -f /./opt/roadsend/pcc/libs/bigloo/2.6f/bigloo.heap
rm -f /./opt/roadsend/pcc/libs/bigloo/2.6f/libbigloo_u-2.6f.so
rm -f /./opt/roadsend/pcc/libs/bigloo/2.6f/libbigloogc-2.6f.so
rm -f /./opt/roadsend/pcc/libs/bigloo/2.6f/libbigloo_u-2.6f.a
rm -f /./opt/roadsend/pcc/libs/bigloo/2.6f/libbigloogc-2.6f.a
rm -f /./opt/roadsend/pcc/libs/php-runtime.heap
rm -f /./opt/roadsend/pcc/libs/php-std.heap
rm -f /./opt/roadsend/pcc/libs/php-xml.heap
rm -f /./opt/roadsend/pcc/libs/phpeval.heap
rm -f /./opt/roadsend/pcc/libs/profiler.heap
rm -f /./opt/roadsend/pcc/libs/webconnect.heap
rm -f /./opt/roadsend/pcc/libs/php-gtk.init
rm -f /./opt/roadsend/pcc/libs/php-xml.init
rm -f /./opt/roadsend/pcc/libs/phpeval.init
rm -f /./opt/roadsend/pcc/libs/libmyapp_s.so
rm -f /./opt/roadsend/pcc/libs/libwebserver.so
rm -f /./opt/roadsend/pcc/libs/libwebserver.a
rm -f /./opt/roadsend/pcc/libs/web_server.h
rm -f /./opt/roadsend/pcc/libs/libbigloo_u-2.6f.so
rm -f /./opt/roadsend/pcc/libs/libbigloogc-2.6f.so
rm -f /./opt/roadsend/pcc/modules/fastcgi/pcc.fcgi
rm -f /./opt/roadsend/pcc/modules/apache1.3.x/mod_pcc.so
rm -f /./opt/roadsend/pcc/bin/bigloo
rm -f /usr/lib/libbigloo_u-2.6f.so
rm -f /usr/lib/libbigloogc-2.6f.so
rm -f /usr/lib/libcgi_u.so
rm -f /usr/lib/libfastcgi_u.so
rm -f /usr/lib/libmhttpd_u.so
rm -f /usr/lib/libmyapp_s.so
rm -f /usr/lib/libphp-curl_u.so
rm -f /usr/lib/libphp-gtk_u.so
rm -f /usr/lib/libphp-mysql_u.so
rm -f /usr/lib/libphp-pcre_u.so
rm -f /usr/lib/libphp-runtime_u.so
rm -f /usr/lib/libphp-std_u.so
rm -f /usr/lib/libphp-xml_u.so
rm -f /usr/lib/libphpeval_u.so
rm -f /usr/lib/libprofiler_u.so
rm -f /usr/lib/libwebconnect_u.so
rm -f /usr/lib/libwebserver.so
rm -f /usr/lib/libcgi_u.a
rm -f /usr/lib/libfastcgi_u.a
rm -f /usr/lib/libmhttpd_u.a
rm -f /usr/lib/libphp-curl_u.a
rm -f /usr/lib/libphp-gtk_u.a
rm -f /usr/lib/libphp-mysql_u.a
rm -f /usr/lib/libphp-pcre_u.a
rm -f /usr/lib/libphp-runtime_u.a
rm -f /usr/lib/libphp-std_u.a
rm -f /usr/lib/libphp-xml_u.a
rm -f /usr/lib/libphpeval_u.a
rm -f /usr/lib/libprofiler_u.a
rm -f /usr/lib/libwebconnect_u.a
rm -f /usr/lib/libwebserver.a
    if [ -f "/sbin/ldconfig" ]; then
        echo "running ldconfig ..."
        /sbin/ldconfig
    fi
rm -f /opt/roadsend/pcc/bin/roadsend-pcc-runtime-uninstall.sh
{% endcodeblock %}

看来还有一些项目要复制或者软连接到/usr/lib/去
```sh
sudo cp /opt/roadsend/pcc/libs/libbigloo_u-2.6f.so /usr/lib/
sudo cp /opt/roadsend/pcc/libs/libbigloogc-2.6f.so /usr/lib/
sudo cp /opt/roadsend/pcc/libs/libcgi_u.so /usr/lib/
sudo cp /opt/roadsend/pcc/libs/libfastcgi_u.so /usr/lib/
sudo cp /opt/roadsend/pcc/libs/libmhttpd_u.so /usr/lib/
sudo cp /opt/roadsend/pcc/libs/libmyapp_s.so /usr/lib/
sudo cp /opt/roadsend/pcc/libs/libphp-curl_u.so /usr/lib/
sudo cp /opt/roadsend/pcc/libs/libphp-gtk_u.so /usr/lib/
sudo cp /opt/roadsend/pcc/libs/libphp-mysql_u.so /usr/lib/
sudo cp /opt/roadsend/pcc/libs/libphp-pcre_u.so /usr/lib/
sudo cp /opt/roadsend/pcc/libs/libphp-runtime_u.so /usr/lib/
sudo cp /opt/roadsend/pcc/libs/libphp-std_u.so /usr/lib/
sudo cp /opt/roadsend/pcc/libs/libphp-xml_u.so /usr/lib/
sudo cp /opt/roadsend/pcc/libs/libphpeval_u.so /usr/lib/
sudo cp /opt/roadsend/pcc/libs/libprofiler_u.so /usr/lib/
sudo cp /opt/roadsend/pcc/libs/libwebconnect_u.so /usr/lib/
sudo cp /opt/roadsend/pcc/libs/libwebserver.so /usr/lib/
sudo cp /opt/roadsend/pcc/libs/libcgi_u.a /usr/lib/
sudo cp /opt/roadsend/pcc/libs/libmhttpd_u.a /usr/lib/
sudo cp /opt/roadsend/pcc/libs/libphp-curl_u.a /usr/lib/
sudo cp /opt/roadsend/pcc/libs/libphp-gtk_u.a /usr/lib/
sudo cp /opt/roadsend/pcc/libs/libphp-mysql_u.a /usr/lib/
sudo cp /opt/roadsend/pcc/libs/libphp-pcre_u.a /usr/lib/
sudo cp /opt/roadsend/pcc/libs/libphp-runtime_u.a /usr/lib/
sudo cp /opt/roadsend/pcc/libs/libphp-std_u.a /usr/lib/
sudo cp /opt/roadsend/pcc/libs/libphp-xml_u.a /usr/lib/
sudo cp /opt/roadsend/pcc/libs/libphpeval_u.a /usr/lib/
sudo cp /opt/roadsend/pcc/libs/libprofiler_u.a /usr/lib/
sudo cp /opt/roadsend/pcc/libs/libwebconnect_u.a /usr/lib/
sudo cp /opt/roadsend/pcc/libs/libwebserver.a /usr/lib/
```

除此之外，还需要复制配置文件

```sh
sudo scp user@hostA:/etc/pcc.conf /etc/
sudo scp user@hostA:/etc/pcc_conf.conf /etc/
```
再次运行
```sh
$ pcc --static index.php -o index

*** ERROR:bigloo:error-handler:
Extension /opt/roadsend/pcc/libs/libphp-pcre_u.so didn't load because: Shared object "libpcre.so.0" not found, required by "libphp-pcre_u.so", dynamic-load:/opt/roadsend/pcc/libs/libphp-pcre_u.so, /opt/roadsend/pcc/libs/libphp-pcre_u.so
You may wish to remove this extension from /etc/pcc.conf if it exists. -- error-handler
```
又出现一个
```sh
sudo scp user@hostA:/usr/local/lib/libpcre.so.0 /usr/local/lib/
sudo scp user@hostA:/usr/local/lib/libgcc_s.so.1 /usr/local/lib/
```
再编译
```
$ pcc --static index.php -o index
Error: problem running command 'gcc', exit status 1
Rerunning with debug level 2 may provide more information.
$ pcc --static index.php -o index -d 2
...
>>>   /usr/bin/ld: cannot find -lcurl
ERR:
Error: problem running command 'gcc', exit status 1
>>>   cleaning up...
```
这个只要curl装好了其实应该就可以了，为什么还会报？根据同事的帮助，可以直接
```sh
sudo cp /usr/local/lib/libcurl.a /lib/
```
这样就肯定能找到了,再次编译运行终于出现php的信息了。
![Alt text](/images/evoup/pcc_result2.png)

告一段落，下一篇写怎么把这些东西做成rpm直接安装
