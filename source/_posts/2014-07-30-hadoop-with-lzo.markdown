---
layout: post
title: "centos上hadoop2.x hdfs支持lzo"
date: 2014-07-30 18:32
comments: true
categories: hadoop
---

总体来说要让hadoop2.x的hdfs支持lzo压缩格式，是和hadoop1.x下配置方式一样的。下文记录一下配置过程。

<!-- more -->

首先安装lzo
```bash
sudo yum install lzo-devel.x86_64 lzop.x86_64
```

下载twitter的hadoop-lzo项目
```bash
git clone https://github.com/twitter/hadoop-lzo.git
```

64位环境的需要设置两个环境变量：
```bash
$ vim ~/.bashrc
export CFLAGS=-m64
export CXXFLAGS=-m64
```

开始编译
```bash
mvn clean package -Dmaven.test.skip=true
```

出现警告
```bash
[WARNING] Javadoc Warnings
[WARNING] /home/hadoop/software/hadoop-lzo/src/main/java/com/hadoop/compression/lzo/LzoIndexer.java:115: 警告 - @return 标记没有参数。
[WARNING] /home/hadoop/software/hadoop-lzo/src/main/java/com/hadoop/compression/lzo/LzoIndexer.java:51: 警告 - @param argument "lzoUri" 不是参数名称。
[INFO] Building jar: /home/hadoop/software/hadoop-lzo/target/hadoop-lzo-0.4.20-SNAPSHOT-javadoc.jar
```
不用管，编译ok。

需要把本地库和jar包放到hadoop对应目录中
```bash
cp target/native/Linux-amd64-64/lib/* $HADOOP_HOME/lib/native/
cp target/hadoop-lzo-0.4.20-SNAPSHOT.jar  $HADOOP_HOME/share/hadoop/mapreduce/lib/
```

3 修改hadoop的配置文件core-site.xml
修改/增加以下2个参数：
```java
io.compression.codecs
org.apache.hadoop.io.compress.GzipCodec,
org.apache.hadoop.io.compress.DefaultCodec,
org.apache.hadoop.io.compress.BZip2Codec,
com.hadoop.compression.lzo.LzoCodec,
com.hadoop.compression.lzo.LzopCodec

io.compression.codec.lzo.class
com.hadoop.compression.lzo.LzoCodec
```
然后重启hadoop看效果。

参考
http://www.linuxidc.com/Linux/2014-05/101090.htm
