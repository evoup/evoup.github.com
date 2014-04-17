---
layout: post
title: "freebsd6.2i386安装subversion"
date: 2013-04-17 14:45
comments: true
categories: [version_control,freebsd]
---

一台老的freebsd6.2机器，port中没有subversion（注：千万不要portsnap更新，一更新后就毁了` Could not find bsd.port.options.mk `没救了，因为最新portsnap不向下兼容支持老系统了！）

注意编译subversion的时候会出现
` configure: error: C++ preprocessor "/lib/cpp" fails sanity check `

<!-- more -->

对g++做个软连接
```sh
sudo ln -s /usr/bin/g++ /usr/local/bin/g++44
```

###安装依赖
```sh
axel http://apache.fayea.com/apache-mirror//apr/apr-1.5.0.tar.gz
tar xzf apr-1.5.0.tar.gz
cd apr-1.5.0
./configure
make
sudo make install
fetch http://mirror.bit.edu.cn/apache//apr/apr-util-1.5.3.tar.gz
tar xjf apr-util-1.5.3.tar.gz
cd apr-util-1.5.3
make 
sudo make install
```

###安装subversion
```sh
fetch http://mirrors.hust.edu.cn/apache/subversion/subversion-1.8.8.tar.gz
tar xzf subversion-1.8.8.tar.gz
cd subversion-1.8.8
fetch http://www.sqlite.org/sqlite-amalgamation-3071501.zip
unzip sqlite-amalgamation-3071501.zip
mv sqlite-amalgamation-3071501 sqlite-amalgamation
./configure --with-apr=/usr/local/apr/ --with-apr-util=/usr/local/apr/
make
sudo make install
```

收工！