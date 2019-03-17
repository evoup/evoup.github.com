---
layout: post
title: "hadoop2.x本地库安装"
date: 2014-07-30 18:00
comments: true
categories: [hadoop]
---
运行hadoop cli时会出现 

` 14/07/25 14:45:25 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform...` 

` using builtin-java classes where applicable `

虽然表面上看来不影响工作结果，但是放着警告不处理太不科学了，想办法解决，过程还是比较漫长，要解决的要有耐心看下文了。
<!-- more -->


###首先查看本地库

```bash
ls $HADOOP_HOME/lib/native/
```

发现动态库不存在,按照上文述及，也会这个错误。
查看系统的libc版本

```bash
$ ll /lib64/libc.so.6
lrwxrwxrwx. 1 root root 12 12月 12 2013 /lib64/libc.so.6 -> libc-2.18.so
```

系统的版本为2.18

参考cdh手册
[NativeLibraries.html#Native_Libraries_Guide](http://archive.cloudera.com/cdh4/cdh/4/hadoop/hadoop-project-dist/hadoop-common/NativeLibraries.html#Native_Libraries_Guide)


找一个hadoop-mapreduce1-project的项目用ant跑一下

```bash
ant -Dcompile.native=true
```

期间会报错jvm-check需要1.6，我的是1.7，直接修改ant的build.xml文件，把里面的1.6改成1.7就可以了。

慢慢等编译完成，报错以下依赖没有解决

```bash
[ivy:resolve] :: resolution report :: resolve 11214ms :: artifacts dl 10ms
        ---------------------------------------------------------------------
        |                  |            modules            ||   artifacts   |
        |       conf       | number| search|dwnlded|evicted|| number|dwnlded|
        ---------------------------------------------------------------------
        |      common      |   10  |   2   |   0   |   0   ||   8   |   0   |
        ---------------------------------------------------------------------
[ivy:resolve]
[ivy:resolve] :: problems summary ::
[ivy:resolve] :::: WARNINGS
[ivy:resolve]           ::::::::::::::::::::::::::::::::::::::::::::::
[ivy:resolve]           ::          UNRESOLVED DEPENDENCIES         ::
[ivy:resolve]           ::::::::::::::::::::::::::::::::::::::::::::::
[ivy:resolve]           :: org.codehaus.jackson#jackson-mapper-asl;1.0.1: several problems occurred while resolving dependency: org.codehaus.jackson#jackson-mapper-asl;1.0.1 {common=[default]}:
[ivy:resolve]   reactor-repo: unable to get resource for org/codehaus/jackson#jackson-mapper-asl;1.0.1: res=${reactor.repo}/org/codehaus/jackson/jackson-mapper-asl/1.0.1/jackson-mapper-asl-1.0.1.pom: java.net.MalformedURLException: no protocol: ${reactor.repo}/org/codehaus/jackson/jackson-mapper-asl/1.0.1/jackson-mapper-asl-1.0.1.pom
[ivy:resolve]   reactor-repo: unable to get resource for org/codehaus/jackson#jackson-mapper-asl;1.0.1: res=${reactor.repo}/org/codehaus/jackson/jackson-mapper-asl/1.0.1/jackson-mapper-asl-1.0.1.jar: java.net.MalformedURLException: no protocol: ${reactor.repo}/org/codehaus/jackson/jackson-mapper-asl/1.0.1/jackson-mapper-asl-1.0.1.jar
[ivy:resolve]           :: org.codehaus.jackson#jackson-core-asl;1.0.1: several problems occurred while resolving dependency: org.codehaus.jackson#jackson-core-asl;1.0.1 {common=[default]}:
[ivy:resolve]   reactor-repo: unable to get resource for org/codehaus/jackson#jackson-core-asl;1.0.1: res=${reactor.repo}/org/codehaus/jackson/jackson-core-asl/1.0.1/jackson-core-asl-1.0.1.pom: java.net.MalformedURLException: no protocol: ${reactor.repo}/org/codehaus/jackson/jackson-core-asl/1.0.1/jackson-core-asl-1.0.1.pom
[ivy:resolve]   reactor-repo: unable to get resource for org/codehaus/jackson#jackson-core-asl;1.0.1: res=${reactor.repo}/org/codehaus/jackson/jackson-core-asl/1.0.1/jackson-core-asl-1.0.1.jar: java.net.MalformedURLException: no protocol: ${reactor.repo}/org/codehaus/jackson/jackson-core-asl/1.0.1/jackson-core-asl-1.0.1.jar
[ivy:resolve]           ::::::::::::::::::::::::::::::::::::::::::::::
[ivy:resolve]
[ivy:resolve] :: USE VERBOSE OR DEBUG MESSAGE LEVEL FOR MORE DETAILS

BUILD FAILED
/home/hadoop/software/hadoop-2.0.0-cdh4.7.0/src/hadoop-mapreduce1-project/build.xml:614: The following error occurred while executing this line:
/home/hadoop/software/hadoop-2.0.0-cdh4.7.0/src/hadoop-mapreduce1-project/src/contrib/build.xml:30: The following error occurred while executing this line:
/home/hadoop/software/hadoop-2.0.0-cdh4.7.0/src/hadoop-mapreduce1-project/src/contrib/build-contrib.xml:440: impossible to resolve dependencies:
        resolve failed - see output for details

Total time: 1 minute 33 seconds
```

少几个jar包，能不能通过直接复制到.ivy2目录的方式解决呢？暂时没这个能力

尝试进到内部工程文件

```bash
cd ./src/contrib/capacity-scheduler/
```

直接ant,报错一样的

原来是要定义reactor.repo这个参数
参考
[《基于hadoop2.0.0的fuse安装以及libhdfs和fuse-dfs的编译》](http://www.verydemo.com/demo_c161_i273713.html)

解决办法： 

   需要定义reactor.repo的url：

  在/usr/hadoop/src/hadoop-mapreduce1-project/ivy/ivysettings.xml文件中添加：

```bash
 <property name="reactor.repo"
           value="http://repo1.maven.org/maven2/"
  override="false"/>
```

添加之后就能找到jackson

但是继续编译会出现

```bash
compile:
     [echo] contrib: gridmix
    [javac] /home/hadoop/software/hadoop-2.0.0-cdh4.7.0/src/hadoop-mapreduce1-project/src/contrib/build-contrib.xml:193: warning: 'includeantruntime' was not set, defaulting to build.sysclasspath=last; set to false for repeatable builds
    [javac] Compiling 31 source files to /home/hadoop/software/hadoop-2.0.0-cdh4.7.0/src/hadoop-mapreduce1-project/build/contrib/gridmix/classes
    [javac] /home/hadoop/software/hadoop-2.0.0-cdh4.7.0/src/hadoop-mapreduce1-project/src/contrib/gridmix/src/java/org/apache/hadoop/mapred/gridmix/Gridmix.java:396: 错误: 类型参数? extends T不在类型变量E的范围内
    [javac]   private <T> String getEnumValues(Enum<? extends T>[] e) {
    [javac]                                         ^
    [javac]   其中, T,E是类型变量:
    [javac]     T扩展已在方法 <T>getEnumValues(Enum<? extends T>[])中声明的Object
    [javac]     E扩展已在类 Enum中声明的Enum<E>
    [javac] /home/hadoop/software/hadoop-2.0.0-cdh4.7.0/src/hadoop-mapreduce1-project/src/contrib/gridmix/src/java/org/apache/hadoop/mapred/gridmix/Gridmix.java:399: 错误: 类型参数? extends T不在类型变量E的范围内
    [javac]     for (Enum<? extends T> v : e) {
    [javac]               ^
    [javac]   其中, T,E是类型变量:
    [javac]     T扩展已在方法 <T>getEnumValues(Enum<? extends T>[])中声明的Object
    [javac]     E扩展已在类 Enum中声明的Enum<E>
    [javac] 注: 某些输入文件使用或覆盖了已过时的 API。
    [javac] 注: 有关详细信息, 请使用 -Xlint:deprecation 重新编译。
    [javac] 注: 某些输入文件使用了未经检查或不安全的操作。
    [javac] 注: 有关详细信息, 请使用 -Xlint:unchecked 重新编译。
    [javac] 2 个错误

BUILD FAILED
/home/hadoop/software/hadoop-2.0.0-cdh4.7.0/src/hadoop-mapreduce1-project/build.xml:614: The following error occurred while executing this line:
/home/hadoop/software/hadoop-2.0.0-cdh4.7.0/src/hadoop-mapreduce1-project/src/contrib/build.xml:30: The following error occurred while executing this line:
/home/hadoop/software/hadoop-2.0.0-cdh4.7.0/src/hadoop-mapreduce1-project/src/contrib/build-contrib.xml:193: Compile failed; see the compiler error output for details.
```

可以参考[《hadoop1.0.3编译eclipse plug-in》](http://www.th7.cn/Program/java/201208/88028.shtml)

所以修改代码把Enum<? extends T>改成Enum<?>
重新用ant编译
编译成功
再用

```bash
ant -Dcompile.native=true
```

看看能否编译出libhadoop.so

编译成功

```bash
find . -type f -name "libhadoop.so"
```

但是没有编译出libhadoop.so？

官方文档误导了（说是ant，在根目录又没有ant的配置文件，根目录下build文件夹的native目录是空的），其实要用maven编译！
其实需要完全编译hadoop才行《hadoop2.2.0 centos 编译安装详解》
http://f.dataguru.cn/thread-245387-1-1.html

```bash
cd /home/hadoop/software/hadoop-2.0.0-cdh4.7.0/src
mvn package -Pdist,native -DskipTests -Dtar
```

报错` [ERROR] Failed to execute goal org.apache.maven.plugins:maven-enforcer-plugin:1.0 `
` :enforce (default) on project hadoop-main: Some Enforcer rules have failed.` 
` Look above for specific messages explaining why the rule failed. -> [Help 1] `

http://zhwj184.iteye.com/blog/1528627
如果为了提高执行速度，不想运行这个插件，则可以通过-Denforcer.skip=true或者简单的-Dskip=true就可以跳过这个这个插件的运行。如果mvn package过程中出现有关这个插件的异常，则可以简单通过这个参数跳过这个验证。

那么直接在编译中加参数-Denforcer.skip=true

```bash
mvn package -Pdist,native -DskipTests -Dtar -Denforcer.skip=true
```

再编译，这么很慢，要下载的文件很多
再次报错 ` [ERROR] Failed to execute goal org.apache.maven.plugins:maven-antrun-plugin:1.6:` 
` run (compile-proto) on project hadoop-common: An Ant `
` BuildException has occured: exec returned: 1 -> [Help 1] `

参考[《hadoop源码编译问题》](http://wenku.baidu.com/link?url=Pkwth1GX3zRyM9BOaxuLhtQWI0dIcWUd7RYtHlvC0b5UpdoS1nc0Xxyn8dhOwnFMytT-Qeo_UZ31WxKE5ruREXF9Al_OGD6E5wr8GB8QRJy)

看来是protoc版本过低导致的

```bash
wget http://protobuf.googlecode.com/files/protobuf-2.4.1.tar.gz
```

被墙的自己想办法

```bash
tar xzf protobuf-2.4.1.tar.gz
cd protobuf-2.4.1
[hadoop@mdn4namenode1 protobuf-2.4.1]$ ./configure
checking build system type... x86_64-unknown-linux-gnu
checking host system type... x86_64-unknown-linux-gnu
checking target system type... x86_64-unknown-linux-gnu
checking for a BSD-compatible install... /usr/bin/install -c
checking whether build environment is sane... yes
checking for a thread-safe mkdir -p... /usr/bin/mkdir -p
checking for gawk... gawk
checking whether make sets $(MAKE)... yes
checking for gcc... no
checking for cc... no
checking for cl.exe... no
configure: error: in `/home/hadoop/software/protobuf-2.4.1':
configure: error: no acceptable C compiler found in $PATH
See `config.log' for more details.

sudo yum install gcc
```

继续安装

` configure: error: C++ preprocessor "/lib/cpp" fails sanity check `

又出现很多问题

```bash
sudo yum install gcc-c++
```

再次

```bash
./configure
make
sudo make install
```

回来继续编译hadoop

```
[ERROR] Failed to execute goal org.apache.maven.plugins:maven-antrun-plugin:1.6:run (make) on project hadoop-common: An Ant BuildException has occured: Execute failed: java.io.IOException: Cannot run program "cmake" (in directory "/home/hadoop/software/hadoop-2.0.0-cdh4.7.0/src/hadoop-common-project/hadoop-common/target/native"): error=2, 没有那个文件或目录 -> [Help 1]
```

没装cmake

```bash
sudo yum install cmake
```

还有一些也装上

```bash
sudo yum install pkgconfig  openssl openssl-devel 
```

这些可以参考《64位操作系统下重新编译hadoop-2.2.0》（这文章提到应该用mvn package -DskipTests -Pdist,native -Dtar来编译hadoop2）
http://blog.itpub.net/20777547/viewspace-1147174

```bash
[INFO]                                                                                                                             [40/1862]
[INFO] Apache Hadoop Main ................................ SUCCESS [5.349s]
[INFO] Apache Hadoop Project POM ......................... SUCCESS [3.432s]
[INFO] Apache Hadoop Annotations ......................... SUCCESS [6.518s]
[INFO] Apache Hadoop Assemblies .......................... SUCCESS [0.875s]
[INFO] Apache Hadoop Project Dist POM .................... SUCCESS [3.334s]
[INFO] Apache Hadoop Auth ................................ SUCCESS [6.069s]
[INFO] Apache Hadoop Auth Examples ....................... SUCCESS [4.573s]
[INFO] Jets3t Cloudera Dependencies ...................... SUCCESS [6.534s]
[INFO] Apache Hadoop Common .............................. SUCCESS [2:01.338s]
[INFO] Apache Hadoop Common Project ...................... SUCCESS [0.452s]
[INFO] Apache Hadoop HDFS ................................ SUCCESS [2:06.706s]
[INFO] Apache Hadoop HttpFS .............................. SUCCESS [24.891s]
[INFO] Apache Hadoop HDFS Project ........................ SUCCESS [0.744s]
[INFO] hadoop-yarn ....................................... SUCCESS [28.902s]
[INFO] hadoop-yarn-api ................................... SUCCESS [1:08.858s]
[INFO] hadoop-yarn-common ................................ SUCCESS [56.000s]
[INFO] hadoop-yarn-server ................................ SUCCESS [0.517s]
[INFO] hadoop-yarn-server-common ......................... SUCCESS [16.093s]
[INFO] hadoop-yarn-server-nodemanager .................... SUCCESS [31.421s]
[INFO] hadoop-yarn-server-web-proxy ...................... SUCCESS [7.293s]
[INFO] hadoop-yarn-server-resourcemanager ................ SUCCESS [22.440s]
[INFO] hadoop-yarn-server-tests .......................... SUCCESS [1.168s]
[INFO] hadoop-yarn-client ................................ SUCCESS [8.536s]
[INFO] hadoop-yarn-applications .......................... SUCCESS [0.276s]
[INFO] hadoop-yarn-applications-distributedshell ......... SUCCESS [4.767s]
[INFO] hadoop-mapreduce-client ........................... SUCCESS [0.433s]
[INFO] hadoop-mapreduce-client-core ...................... SUCCESS [54.060s]
[INFO] hadoop-yarn-applications-unmanaged-am-launcher .... SUCCESS [5.197s]
[INFO] hadoop-yarn-site .................................. SUCCESS [0.517s]
[INFO] hadoop-yarn-project ............................... SUCCESS [36.840s]
[INFO] hadoop-mapreduce-client-common .................... SUCCESS [33.592s]
[INFO] hadoop-mapreduce-client-shuffle ................... SUCCESS [5.179s]
[INFO] hadoop-mapreduce-client-app ....................... SUCCESS [18.536s]
[INFO] hadoop-mapreduce-client-hs ........................ SUCCESS [9.267s]
[INFO] hadoop-mapreduce-client-jobclient ................. SUCCESS [10.860s]
[INFO] hadoop-mapreduce-client-hs-plugins ................ SUCCESS [4.425s]
[INFO] Apache Hadoop MapReduce Examples .................. SUCCESS [14.302s]
[INFO] hadoop-mapreduce .................................. SUCCESS [4.028s]
[INFO] Apache Hadoop MapReduce Streaming ................. SUCCESS [9.253s]
[INFO] Apache Hadoop Distributed Copy .................... SUCCESS [1:14.406s]
[INFO] Apache Hadoop Archives ............................ SUCCESS [4.108s]
[INFO] Apache Hadoop Rumen ............................... SUCCESS [11.244s]
[INFO] Apache Hadoop Gridmix ............................. SUCCESS [10.038s]
[INFO] Apache Hadoop Data Join ........................... SUCCESS [5.673s]
[INFO] Apache Hadoop Extras .............................. SUCCESS [6.033s]
[INFO] Apache Hadoop Pipes ............................... SUCCESS [14.666s]
[INFO] Apache Hadoop Tools Dist .......................... SUCCESS [3.519s]
[INFO] Apache Hadoop Tools ............................... SUCCESS [0.066s]
[INFO] Apache Hadoop Distribution ........................ SUCCESS [30.039s]
[INFO] Apache Hadoop Client .............................. SUCCESS [20.618s]
[INFO] Apache Hadoop Mini-Cluster ........................ SUCCESS [2.427s]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 15:51.852s
[INFO] Finished at: Fri Jul 25 17:40:23 CST 2014
[INFO] Final Memory: 127M/305M
[INFO] ------------------------------------------------------------------------
```

全部编译成功，然后找下libhadoop.so

```bash
$ find . -name "libhadoop.so"
./src/hadoop-common-project/hadoop-common/target/hadoop-common-2.0.0-cdh4.7.0/lib/native/libhadoop.so
./src/hadoop-common-project/hadoop-common/target/native/target/usr/local/lib/libhadoop.so
./src/hadoop-dist/target/hadoop-2.0.0-cdh4.7.0/lib/native/libhadoop.so
```

编译出来了，然后放到文章一开始说的$HADOOP_HOME/lib/native/目录下面，以后运行就不再报错了。


参考文章
<a href="http://www.linuxidc.com/Linux/2012-04/59200.htm">《Hadoop本地库与系统版本不一致引起的错误解决方法》</a>
