---
layout: post
title: "freebsd6.2i386安装subversion1.8.8"
date: 2013-04-17 14:45
comments: true
categories: [version_control,freebsd]
---

一台老的freebsd6.2机器，port中没有subversion（注：千万不要portsnap更新，一更新后就毁了` Could not find bsd.port.options.mk `没救了，因为最新portsnap不向下兼容支持老系统了！）

注意编译subversion的时候会出现
` configure: error: C++ preprocessor "/lib/cpp" fails sanity check `

<!-- more -->

对g++做个软连接

```bash
$ sudo ln -s /usr/bin/g++ /usr/local/bin/g++44
```

###安装依赖

```bash
$ cd ~/software/
$ axel http://apache.fayea.com/apache-mirror//apr/apr-1.5.0.tar.gz
$ tar xzf apr-1.5.0.tar.gz
$ cd apr-1.5.0
$ ./configure
$ make
$ sudo make install
$ fetch http://mirror.bit.edu.cn/apache//apr/apr-util-1.5.3.tar.gz
$ tar xjf apr-util-1.5.3.tar.gz
$ cd apr-util-1.5.3
$ make 
$ sudo make install
```

###安装subversion

```bash
$ cd ~/software
$ fetch http://mirrors.hust.edu.cn/apache/subversion/subversion-1.8.8.tar.gz
$ tar xzf subversion-1.8.8.tar.gz
$ cd subversion-1.8.8
$ fetch http://www.sqlite.org/sqlite-amalgamation-3071501.zip
$ unzip sqlite-amalgamation-3071501.zip
$ mv sqlite-amalgamation-3071501 sqlite-amalgamation
$ ./configure --with-apr=/usr/local/apr/ --with-apr-util=/usr/local/apr/
$ make
$ sudo make install
```

这样是可以用基本的svn签出提交了，但是后来又在签出代码时发现了svn e170000 unrecognized url scheme for http（https）的问题。
原来少安装一个模块serf（1.8版本之前用neon），安装次序分别为python2.7->scons2.3.1->serf1.3.4

```bash
$ cd ~/software
$ fetch https://www.python.org/ftp/python/2.7/Python-2.7.tgz
$ tar xzf Python-2.7
$ cd Python-2.7
$ ./configure
$ make
$ sudo make install
$ fetch http://cznic.dl.sourceforge.net/project/scons/scons/2.3.1/scons-2.3.1.tar.gz
$ tar xzf scons-2.3.1.tar.gz
$ cd scons-2.3.1
$ sudo python setup.py install --standard-lib
$ fetch http://serf.googlecode.com/svn/src_releases/serf-1.3.4.tar.bz2
$ tar xjf serf-1.3.4.tar.bz2
$ cd serf-1.3.4
$ scons APR=/usr/local/apr/ APU=/usr/local/apr/
$ sudo scons install
scons: Reading SConscript files ...
scons: done reading SConscript files.
scons: Building targets ...
Install file: "libserf-1.a" as "/usr/local/lib/libserf-1.a"
Install file: "libserf-1.so.1.3.0" as "/usr/local/lib/libserf-1.so.1.3.0"
Install file: "serf.h" as "/usr/local/include/serf-1/serf.h"
Install file: "serf_bucket_types.h" as "/usr/local/include/serf-1/serf_bucket_types.h"
Install file: "serf_bucket_util.h" as "/usr/local/include/serf-1/serf_bucket_util.h"
Install file: "serf-1.pc" as "/usr/local/lib/pkgconfig/serf-1.pc"
scons: done building targets.
```

重新安装subversion，把openssl也带上

```bash
$ cd ../subversion-1.8.8
$ ./configure --with-apr=/usr/local/apr/ --with-apr-util=/usr/local/apr/ --with-openssl --with-serf=/usr/local/
```

如果遇到找不到libintl.h

```bash
$ sudo ln -s /usr/local/include/libintl.h /usr/include/
```

然后编译

```bash
make
sudo make install
```

收工

