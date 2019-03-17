---
layout: post
title: "SpringMVC搭建篇"
date: 2014-02-12 15:49
comments: true
categories: [java]
---

最近有项目要求必须用java，于是想到了spring，前几天花了一天时间也算调通。这里简单记录下先(其实很详细，我自己要看，哈哈)，其实更喜欢idea，人家都是google官方android开发推荐ide，有专门插件。嗯，扯远了，下面是搭建的过程小记：

<!-- more -->
###Spring是什么？
不废话了，自己去查--

###搭建的环境
windows7+eclipse kepler sr2+jdk1.7+tomcat7.0

第一个spring程序的代码地址（我借鉴的，也可以自己改改呗:)）：
https://github.com/evoup/javaweb.git
签出后我把它放到了c:/code_template文件夹下。
![Alt text](/images/evoup/springmvc_first/git_clone_helloworld.jpg)


####eclipse
首先自然是下载eclipse了，注意版本4.3.2是优化过的版本，4.3.1（SR1）这个版本不是很建议，因为搭建的时候貌似不是很顺利？

32位的下载地址：
http://www.eclipse.org/downloads/download.php?file=/technology/epp/downloads/release/kepler/SR2/eclipse-standard-kepler-SR2-win32.zip

64位的下载地址：
http://www.eclipse.org/downloads/download.php?file=/technology/epp/downloads/release/kepler/SR2/eclipse-standard-kepler-SR2-win32-x86_64.zip

下完放到任意目录，我的是C:\web，直接解压出来
![Alt text](/images/evoup/springmvc_first/eclipse_version.jpg)


####jdk的安装
就用常用的1.7吧
http://www.oracle.com/technetwork/java/javase/downloads/jdk7-downloads-1880260.html

####导入项目

打开eclipse，首先选择工作区
![Alt text](/images/evoup/springmvc_first/select_workspace.jpg)

接着选择菜单
import->Existing Projects into Workspace
![Alt text](/images/evoup/springmvc_first/import.jpg)

选择之前的项目导入
![Alt text](/images/evoup/springmvc_first/import2.jpg)

右键项目属性，转为dynamic web module项目
![Alt text](/images/evoup/springmvc_first/project_property.jpg)


项目 —> 属性 -> Deployment Assembly -> Add -> folder -> 选择src/main/webapp
这一步就是配置webapp目录要发布到项目的根目录下


项目 —> 属性 -> Deployment Assembly -> Add -> Java Build Path Entries -> 选择Maven Dependencies -> Finish -> OK
完了貌似必须重启以下eclipse kepler sr2
还有一点小问题

把头上这段去掉
<!DOCTYPE web-app PUBLIC
 "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
 "http://java.sun.com/dtd/web-app_2_3.dtd" >
 
 好了吧？还木有！需要用tomcat8伪装成tomcat7
 http://blog.csdn.net/yuzhiyuxia/article/details/26282767
 


 
