---
layout: post
title: "编译cdh47下hive的contrib源码"
date: 2014-07-31 10:18
comments: true
categories: [hadoop,hive,java]
---


进入hive的src/contrib目录，开始编译


需要这几个包，加到classpath中去
```
/home/hadoop/software/hadoop-2.0.0-cdh4.7.0/share/hadoop/common/hadoop-common-2.0.0-cdh4.7.0.jar
/home/hadoop/software/hadoop-2.0.0-cdh4.7.0/share/hadoop/mapreduce1/hadoop-core-2.0.0-mr1-cdh4.7.0.jar
```


看下工程路径build.xml的29行
```
<import file="../build-common.xml"/>
```

编辑上级目录的build-common.xml文件
根据这篇文章《如何在Ant中引入第三方Jar包》
http://www.sadtojoy.com/aspx/Detail.aspx?id=4131
找这个文件的关键字classpath和javac，有
```
266   <path id="classpath">
267     <pathelement location="${build.dir.hive}/service/classes"/>
268     <pathelement location="${build.dir.hive}/common/classes"/>
269     <pathelement location="${build.dir.hive}/serde/classes"/>
270     <pathelement location="${build.dir.hive}/metastore/classes"/>
271     <pathelement location="${build.dir.hive}/ql/classes"/>
272     <pathelement location="${build.dir.hive}/beeline/classes"/>
273     <pathelement location="${build.dir.hive}/cli/classes"/>
274     <pathelement location="${build.dir.hive}/shims/classes"/>
275     <pathelement location="${build.dir.hive}/hwi/classes"/>
276     <pathelement location="${build.dir.hive}/jdbc/classes"/>
277     <pathelement location="${build.dir.hive}/hbase-handler/classes"/>
278 <!--\{\{\{把缺少的hadoop加入-->
279     <pathelement location="/home/hadoop/software/hadoop-2.0.0-cdh4.7.0/share/hadoop/common/hadoop-common-2.0.0-cdh4.7.0.jar"/>
280     <pathelement location="/home/hadoop/software/hadoop-2.0.0-cdh4.7.0/share/hadoop/mapreduce1/hadoop-core-2.0.0-mr1-cdh4.7.0.jar"/>
281 <!--\}\}\}-->
282     <fileset dir="${basedir}" includes="lib/*.jar"/>
283     <path refid="common-classpath"/>
284   </path>
```

像上面这样把2个包加入。

然后回到工程文件，执行ant，提示将编译contrib包，一路过去，没有问题
，最后编译的jar包存到了ivy的路径下
```
compile:
     [echo] Project: contrib
    [javac] Compiling 39 source files to /home/hadoop/software/hive-0.10.0-cdh4.7.0/src/build/contrib/classes
    [javac] /home/hadoop/software/hadoop-2.0.0-cdh4.7.0/share/hadoop/common/hadoop-common-2.0.0-cdh4.7.0.jar(org/apache/hadoop/fs/Path.class): 警告: 无法找到类型 'LimitedPrivate' 的注释方法 'value()': 找不到org.apache.hadoop.classification.InterfaceAudience的类文件
    [javac] 注: 某些输入文件使用或覆盖了已过时的 API。
    [javac] 注: 有关详细信息, 请使用 -Xlint:deprecation 重新编译。
    [javac] 注: /home/hadoop/software/hive-0.10.0-cdh4.7.0/src/contrib/src/java/org/apache/hadoop/hive/contrib/udf/example/UDFExampleStructPrint.java使用了未经检查或不安全的操作。
    [javac] 注: 有关详细信息, 请使用 -Xlint:unchecked 重新编译。
    [javac] 1 个警告
     [copy] Warning: /home/hadoop/software/hive-0.10.0-cdh4.7.0/src/contrib/src/java/conf does not exist.

jar:
     [echo] Project: contrib
      [jar] Building jar: /home/hadoop/software/hive-0.10.0-cdh4.7.0/src/build/contrib/hive-contrib-0.10.0-cdh4.7.0.jar
[ivy:publish] :: delivering :: org.apache.hive#hive-contrib;0.10.0-cdh4.7.0 :: 0.10.0-cdh4.7.0 :: integration :: Fri Jul 25 10:54:42 CST 2014
[ivy:publish]   delivering ivy file to /home/hadoop/software/hive-0.10.0-cdh4.7.0/src/build/contrib/ivy-0.10.0-cdh4.7.0.xml
[ivy:publish] :: publishing :: org.apache.hive#hive-contrib
[ivy:publish]   published hive-contrib to /home/hadoop/.ivy2/local/org.apache.hive/hive-contrib/0.10.0-cdh4.7.0/jars/hive-contrib.jar
[ivy:publish]   published ivy to /home/hadoop/.ivy2/local/org.apache.hive/hive-contrib/0.10.0-cdh4.7.0/ivys/ivy.xml

BUILD SUCCESSFUL
Total time: 19 seconds
```
ok
这样就得到了hive-contrib.jar文件,解包看一下内容。没问题，都包含了。