---
layout: post
title: "hiveserver2的maven配置，使用cdh版本"
date: 2014-08-30 17:52
comments: true
categories: [hive,maven]
---

hadoop2.0下开发软件我目前主要采用maven做项目构建管理，但要注意要使生产环境的版本和开发的版本一致。可以做的就是先了解生产环境的版本，然后给自己的软件指定相同的版本，避免兼容问题的发生。以下是方法记录。
<!-- more -->

####添加仓库
首先在pom.xml文件中添加repository

{% codeblock lang:xml pom.xml %}
<repositories>
    <repository>
        <id>cloudera</id>
        <url>https://repository.cloudera.com/artifactory/cloudera-repos/</url>
    </repository>
</repositories>
{% endcodeblock %}

####加入依赖
然后改用cdh的依赖，提供的artifactId和version就可以了
主要是2个包的事情

{% codeblock lang:xml pom.xml %}
<dependency>
    <groupId>org.apache.hive</groupId>
    <artifactId>hive-jdbc</artifactId>
    <version>0.10.0-cdh4.7.0</version>
</dependency>
<dependency>
    <groupId>org.apache.hadoop</groupId>
    <artifactId>hadoop-common</artifactId>
    <version>2.0.0-cdh4.7.0</version>
</dependency>
{% endcodeblock %}


依赖的部分，也可以用以下的方式载入
{% codeblock lang:xml pom.xml %}
<dependency>  
            <groupId>org.apache.hive</groupId>  
            <artifactId>htmlparser</artifactId>  
            <version>2.0</version>  
            <scope>system</scope>  
            <systemPath>${project.basedir}/lib/xxx.jar</systemPath>  
</dependency>  
{% endcodeblock %}

然后是hiveserver2使用和hiveserver1要注意的地方

hiveserver2

```java
URLHIVE="jdbc:hive2://"+hive_host+":"+hive_port+"/smartmad;auth=noSasl";
Class.forName("org.apache.hive.jdbc.HiveDriver");
```

hiveserver1

```java
URLHIVE="jdbc:hive://"+hive_host+":"+hive_port+"/smartmad;auth=noSasl";
Class.forName("org.apache.hadoop.hive.jdbc.HiveDriver");
```

这里我们从hive改进hiveserver2的包命名的角度不难看出，该版本已成为apache顶级项目。


####参考文章
http://www.cloudera.com/content/cloudera-content/cloudera-docs/CDH4/4.2.0/CDH4-Installation-Guide/cdh4ig_topic_31.html
