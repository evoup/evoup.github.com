---
layout: post
title: "maven使用小计"
date: 2014-06-12 14:41
comments: true
categories: [java]
---

###介绍
maven是java的一个比较有名的项目构建工具，类似的工具还有ant、gradle等，采用vim+maven构建java项目需要了解maven的基本操作，以下是项目收集到的一些使用套路和细节注意点。

<!-- more -->

###安装(其实无需安装)

由于可能地址被墙，在国内maven下载依赖比较慢，最好在编辑/etc/hosts，并在其中加上依赖服务器的IP地址，如下：
```
199.27.77.129 repo.maven.apache.org
```

下载和解压安装包
```
mkdir /home/software/
fetch http://down1.chinaunix.net/distfiles/apache-maven-3.0.4-bin.tar.gz
tar xzf apache-maven-3.0.4-bin.tar.gz
```

然后在~/.cshrc中指定(我这里是使用tcsh作为shell环境)
```
setenv M2_HOME "/usr/home/software/apache-maven-3.0.4"
setenv MAVEN_OPTS "-Xms128m -Xmx512m"
set path=( $path $M2_HOME/bin )
```
接着
```
source ~/.cshrc
```
就可以使用mvn命令了

###基本测试构建

生成测试
```
mvn clean test
```
以上命令能直接生成class文件，并且支持自动测试（需要引入Junt并编写测试代码）

生成测试和打包
```
mvn clean package
```
以上命令能直接生成最终的jar包文件，同时实现mvn clean test的所有功能


###常用命令
编译
```
mvn compile
```

清理（删除所有test，package，compile等生成的文件）
```
mvn clean
```

获取所有依赖并且打包
```
mvn dependency:copy-dependencies -DoutputDirectory=lib package
```

不经过测试直接打包
```
mvn -DskipTests clean package
```


###找依赖的方法
以下两个网站任意一个直接在搜索框中给出java根类即可，第一个网站还可以在线查看jar包里的方法，可以当在线手册查阅：

https://repository.sonatype.org/

http://mvnrepository.com/

###创建骨架
```
mvn archetype:create -DgroupId=com.mycompany.app -DartifactId=myapp
```

参考：
《Maven实战》
