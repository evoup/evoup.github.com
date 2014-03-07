---
layout: post
title: "centos下hadoop0.20的LZO压缩支持"
date: 2014-03-07 11:00
comments: true
categories: 
---

###首先安装lzo库
```sh
wget http://www.oberhumer.com/opensource/lzo/download/lzo-2.04.tar.gz
tar xzf lzo-2.04.tar.gz
cd lzo-2.04
./configure --enable-shared
make
sudo make install
```
安装完成后拷贝/usr/local/lib目录下的lzo库文件到/usr/lib(32位平台)或者/usr/lib64(64位平台)

<!-- more -->

```sh
sudo cp /usr/local/lib/liblzo2* /usr/lib
```
ps：应该是这些文件
```
liblzo2.a
liblzo2.la
liblzo2.so
liblzo2.so.2
liblzo2.so.2.0.0
```
在/etc/ld.so.conf.d/目录下新建lzo.conf文件，写入lzo库文件的路径/usr/local/lib，然后运行/sbin/ldconfig -v，使配置生效。

###然后安装hadoop-LZO
我这里是hadoop0.20版本，就下kevinweil的版本
git clone https://github.com/kevinweil/hadoop-lzo

然后安装ant
/etc/profile中
```bash
export ANT_HOME=/home/hadoop/apache-ant-1.9.3
PATH=$ANT_HOME/bin:$PATH
export PATH
```
然后source /etc/profile

然后指定好CFLAGS和CXXFLAGS开始编译

####32位os
```bash
export CFLAGS=-m32
export CXXFLAGS=-m32
ant compile-native tar
```
####64位os
```bash
export CFLAGS=-m64
export CXXFLAGS=-m64
ant compile-native tar
```
等待编译完成
```bash
tar:
      [tar] Building tar: /home/hadoop/project/hadoop-lzo/build/hadoop-lzo-0.4.15.tar.gz

BUILD SUCCESSFUL
Total time: 23 seconds
```
然后把本地库和jar包拷贝到hadoop的目录下，并分发到各节点
```bash
cd build
cp hadoop-lzo-0.4.15.jar /u01/app/hadoop/lib
cp -r native/Linux-i386-32/* /u01/app/hadoop/lib/native/Linux-i386-32/
#如果是64位应该是Linux-amd64-64
```

###安装shell工具lzop
```bash
wget http://www.lzop.org/download/lzop-1.03.tar.gz
./configure
make
sudo make install
```
使用
```bash
#压缩
lzop test.log
#查看解压内容
lzop -cd test.log.lzo | less
#解压
lzop -x test.log.lzo
```


###配置hadoop的core-site.xml和mapred-site.xml
core-site.xml
```
<property>
        <name>io.compression.codecs</name> <value>org.apache.hadoop.io.compress.DefaultCodec,org.apache.hadoop.io.compress.GzipCodec,org.apache.hadoop.io.compress.BZip2Codec,com.hadoop.compression.lzo.LzoCodec,com.hadoop.compression.lzo.LzopCodec</value>
        <!-- 配置 Hadoop 压缩包 -->
</property>
<property> 
        <name>io.compression.codec.lzo.class</name> 
        <value>com.hadoop.compression.lzo.LzoCodec</value> 
</property>
```
mapred-site.xml
```
<property>
      <name>mapred.compress.map.output</name>
      <value>true</value>
      <!-- map 和 reduce 输出中间文件默认开启压缩 -->
  </property>
  <property>      
         <name>mapred.map.output.compression.codec</name>       
         <value>com.hadoop.compression.lzo.LzoCodec</value>      
  </property> 
  <property>
         <name>mapred.child.env</name> 
         <value>LD_LIBRARY_PATH=/u01/app/hadoop/lib/native/Linux-i386-32</value> 
 </property>
```

###启动hadoop并测试lzo

用lzop压缩一个文件，然后上传到hdfs
```
bin/start-all.sh
lzop test.txt
/u01/app/hadoop/bin/hadoop -put test.txt.lzo /testlzo/test.txt.lzo
```

然后测试lzo是否已经支持

在hadoop的conf的hadoop-env.sh中指定好HADOOP_CLASSPATH,然后就不用再
```sh
bin/hadoop jar lib/hadoop-lzo-0.4.15.jar com.hadoop.compression.lzo.LzoIndexer /testlzo/test.txt.lzo
```
而直接使用
```sh
bin/hadoop com.hadoop.compression.lzo.LzoIndexer /testlzo/test.log.lzo
```

报错
> 14/03/06 18:37:14 ERROR lzo.GPLNativeCodeLoader: Could not load native gpl library
> java.lang.UnsatisfiedLinkError: no gplcompression in java.library.path
>        at java.lang.ClassLoader.loadLibrary(ClassLoader.java:1738)
>        at java.lang.Runtime.loadLibrary0(Runtime.java:823)
>        at java.lang.System.loadLibrary(System.java:1028)
>        at com.hadoop.compression.lzo.GPLNativeCodeLoader.<clinit>(GPLNativeCodeLoader.java:32)
>        at com.hadoop.compression.lzo.LzoCodec.<clinit>(LzoCodec.java:71)
>        at com.hadoop.compression.lzo.LzoIndexer.<init>(LzoIndexer.java:36)
>        at com.hadoop.compression.lzo.LzoIndexer.main(LzoIndexer.java:134)
> 14/03/06 18:37:14 ERROR lzo.LzoCodec: Cannot load native-lzo without native-hadoop
> 14/03/06 18:37:14 INFO lzo.LzoIndexer: [INDEX] LZO Indexing file /testlzo/x.log.lzo, size 0.00 GB...
> Exception in thread "main" java.lang.RuntimeException: native-lzo library not available
>        at com.hadoop.compression.lzo.LzopCodec.createDecompressor(LzopCodec.java:104)
>        at com.hadoop.compression.lzo.LzoIndex.createIndex(LzoIndex.java:229)
>        at com.hadoop.compression.lzo.LzoIndexer.indexSingleFile(LzoIndexer.java:117)
>        at com.hadoop.compression.lzo.LzoIndexer.indexInternal(LzoIndexer.java:98)
>        at com.hadoop.compression.lzo.LzoIndexer.index(LzoIndexer.java:52)
>        at com.hadoop.compression.lzo.LzoIndexer.main(LzoIndexer.java:137)



看似是GPLNativeCodeLoader没有装，其实是路径不对
```
[hadoop@mdn2 lib]$ls -la /u01/app/hadoop/lib/native/Linux-i386-32/lib
total 184
drwxrwxr-x 2 hadoop hadoop  4096 Mar  6 14:10 .
drwxrwxrwx 5 hadoop hadoop  4096 Mar  7 10:52 ..
-rw-r--r-- 1 hadoop hadoop 76502 Mar  6 14:10 libgplcompression.a
-rw-rw-r-- 1 hadoop hadoop  1128 Mar  6 14:10 libgplcompression.la
lrwxrwxrwx 1 hadoop hadoop    26 Mar  6 14:10 libgplcompression.so -> libgplcompression.so.0.0.0
lrwxrwxrwx 1 hadoop hadoop    26 Mar  6 14:10 libgplcompression.so.0 -> libgplcompression.so.0.0.0
-rwxrwxr-x 1 hadoop hadoop 59281 Mar  6 14:10 libgplcompression.so.0.0.0
```
其实只要把这几个库复制到上级目录即可
```
cp /u01/app/hadoop/lib/native/Linux-i386-32/lib/* /u01/app/hadoop/lib/native/Linux-i386-32/
```
然后
```
[hadoop@mdn2 hadoop]$bin/hadoop com.hadoop.compression.lzo.LzoIndexer /testlzo/x.log.lzo
14/03/07 10:52:33 INFO lzo.GPLNativeCodeLoader: Loaded native gpl library
14/03/07 10:52:33 INFO lzo.LzoCodec: Successfully loaded & initialized native-lzo library [hadoop-lzo rev 6bb1b7f8b9044d8df9b4d2b6641db7658aab3cf8]
14/03/07 10:52:34 INFO lzo.LzoIndexer: [INDEX] LZO Indexing file /testlzo/x.log.lzo, size 0.00 GB...
14/03/07 10:52:34 INFO lzo.LzoIndexer: Completed LZO Indexing in 0.15 seconds (0.00 MB/s).  Index size is 0.01 KB.
```
终于成功。

参考：

《hadoop安装lzo》
http://blog.csdn.net/wanglinzi/article/details/12318743

《lzo安装说明(2)》
http://www.350351.com/jiagoucunchu/Hadoop/83863_2.html


