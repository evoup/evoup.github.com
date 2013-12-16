---
layout: post
title: "centos安装erlang R16b03"
date: 2013-12-12 16:43
comments: true
categories: erlang
---
###起因
原本以为centos上简单敲几个yum install erlang就可以完成，结果事与愿违，没有package，那么只要乖乖下载源码编译，记录若干步骤备查。

<!-- more -->

###首先安装必要的依赖
```sh
$ sudo yum groupinstall Base "Development Tools" "Perl Support"  
$ sudo yum install gcc glibc-devel make ncurses-devel openssl-devel autoconf
$ sudo yum install unixODBC unixODBC-devel
$ wget http://www.erlang.org/download/otp_src_R16B03.tar.gz
```

###源码下载
并且下载erlang的源码，我这里选择的是最新的R16B03
```sh
$ tar xzf otp_src_R16B03.tar.gz
```

###安装
注意,如果没有先
```sh
$ sudo ln -s /usr/bin/gcc /usr/local/bin/gcc44
```
会收到` checking for C compiler default output file name... configure: error: C compiler cannot create

的错误

```sh
$ cd otp_src_R16B03
$ ./configure --prefix=/home/erlang --without-javac //不用装java
make
$ sudo make install
```
等待安装完成。


###设置环境变量
```sh
$ vim ~/.profile
```
或者
```sh
$ sudo vim /etc/profiles
```
加入
```bash
export PATH=$PATH:/home/erlang/bin
```
加完如果要立即生效忘记去source /etc/profile和source ~/.bash_profile，否则要等到下次登录再可以。

(ps: bash是根据/etc/profile、~/.bash_profile、~/.bash_login或~/.profile的顺序进行读取的。)

(ADDED IN 2013-12-16,今日发现以上代码必须加到~/.bashrc中才行，否则重启后还是不予加载原因未明)

###测试
接下来输入erl,应该就可以看见erlang的提示符了。
```sh
Eshell V5.10.4  (abort with ^G)
1> 
```
写一个test.erl测试
```erlang
-module(test).
-export([sayhello/0]).
sayhello()->
  io:format("hello world~n").
```
编译运行
```sh
erlc test.erl
erl -noshell -s test sayhello -s init stop
hello world
```

ok，完成收工，还不算太复杂，比freebsd上安装顺利点:)
