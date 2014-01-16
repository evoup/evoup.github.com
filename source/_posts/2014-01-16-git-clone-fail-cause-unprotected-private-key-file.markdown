---
layout: post
title: "git clone失败警告unprotected private key file"
date: 2014-01-16 23:41
comments: true
categories: version_control 
---
###起因
今天在clone项目的时候发生了很奇怪的错误，而且居然是有的项目可以clone，有的不行，都是git协议。错误如下：

<!-- more -->
```sh
$ git clone git@github.com:someone/case.git
Cloning into 'case'...
Warning: Permanently added the RSA host key for IP address '192.30.252.129' to t
he list of known hosts.
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@         WARNING: UNPROTECTED PRIVATE KEY FILE!          @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
Permissions 0644 for '/c/Users/Administrator/.ssh/id_rsa' are too open.
It is recommended that your private key files are NOT accessible by others.
This private key will be ignored.
bad permissions: ignore key: /c/Users/Administrator/.ssh/id_rsa
Permission denied (publickey).
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
```
###排查
显示是私钥权限不正确导致的，然后定位到git-bash所使用的sh.exe程序，用管理员模式运行
```sh
chmod 0600 /c/Users/Administrator/.ssh/id_rsa
```
尝试几次都没法收回group和other的r权限（r+w+rw=2^2+2^1+2^0，我这里是4，2^2所以是r）
最后发现是因为装git windows的时候选择了第一种方式和cygwin正好冲突了，如果装了cygwin的基础上再安装git windows要在git windows的安装选项里选第二种方式；还有一种可能是和tortoise git冲突了，尚未确认。
最后删除cygwin重装git-bash可以正常clone了。

###延伸
git windows还有第三种安装方式，第三种比较危险，会覆盖windows指令，一般不采取，如果以后不打算用cygwin的话，直接第一种方式安装就可以了。

附：正常的权限掩码，来自一台freebsd
```sh
-rw-r--r--   authorized_keys
-rw-------   config
-rw-------   id_dsa
-rw-------   id_dsa.pub
-rw-------   id_rsa
-rw-r--r--   id_rsa.pub
-rw-------   known_hosts
```






