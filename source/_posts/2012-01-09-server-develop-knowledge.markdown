---
layout: post
title: "服务器开发知识汇总2010年整理的"
date: 2012-01-09 16:41
comments: true
categories: [socket server]
---

整理了一些服务开发的文章，以备不时之需


一个开源CGI程序
http://www.raosoft.com/help/cgi/ezs/index.html

nginx内存池用法(分析得不透，没有说出父子内存池，也搞错了子内存池其实是放一个指针到父内存池)
http://www.cnblogs.com/sld666666/archive/2010/06/27/1766255.html

<!-- more -->

nginx中的output 
http://simohayha.javaeye.com/blog/651779

nginx的内存管理
http://simohayha.javaeye.com/blog/545192

typedef和函数指针（nginx实现，自己实现过一个）
http://www.cnblogs.com/sld666666/archive/2010/06/20/1761457.html

syslog用法，只要看/var/log/debug的信息就可以了。《Linux下syslog日志函数使用》
http://blog.csdn.net/telehiker/archive/2007/10/18/1830575.aspx

如何改进winsocks版的police server,(参考一goto)
http://www.boluor.com/linux-cs-ipc-using-named-pipe.html

autoconf和automake来生成Makefile
http://www.ibm.com/developerworks/cn/linux/l-makefile/

通用链表（封装得很烂，试过一把）
http://blog.csdn.net/eroswang/archive/2007/07/27/1711369.aspx

线程里处理信号
http://blog.csdn.net/eroswang/archive/2007/07/02/1675740.aspx

HTTP协议中的Tranfer-Encoding：chunked编码解析
http://blog.csdn.net/eroswang/archive/2008/01/16/2047701.aspx

php正则表达参考(转的时候要知道)
http://blog.csdn.net/eroswang/archive/2007/10/05/1812419.aspx

结合这篇一起看
http://deerchao.net/tutorials/regex/regex.htm

线程里sleep的正确的方法（用在定时做服务器广播的那个线程）
http://blog.csdn.net/eroswang/archive/2008/09/15/2932630.aspx

socket封包和拆包
http://blog.csdn.net/eroswang/archive/2008/09/02/2869097.aspx

offsetof和POD
http://hi.baidu.com/infant/blog/item/4439b2fb5ef67e284e4aeac8.html

memcache测试
http://blog.csdn.net/eroswang/archive/2010/07/09/5722669.aspx

fork两次如何避免僵尸进程
http://blog.csdn.net/eroswang/archive/2008/11/19/3333617.aspx

glib库hash表GHashTable介绍(也抄了一个snort的实现，这个是内置的选择)
http://blog.csdn.net/eroswang/archive/2008/12/11/3497556.aspx

Linux动态库(.so)搜索路径(编译时候确认)
http://blog.csdn.net/eroswang/archive/2008/12/30/3645552.aspx

惊群（不要试图把accept加锁,最好连锁都不用）
http://blog.csdn.net/eroswang/archive/2008/12/19/3560366.aspx

攻击工具(h00lyshit.c)
http://www.xfocus.net/tools/12.html

netstat全部参数
http://blog.csdn.net/eroswang/archive/2008/11/17/3322023.aspx

GDB调试在看一次
http://blog.csdn.net/eroswang/archive/2009/03/29/4033722.aspx

使用setsockopt来控制connect超时
http://blog.csdn.net/eroswang/archive/2009/11/17/4819444.aspx

八大排序算法总结
http://blog.csdn.net/eroswang/archive/2009/10/26/4727644.aspx

atoi源码(做地图的时候用的到)
http://blog.csdn.net/eroswang/archive/2010/08/11/5804244.aspx

异步下socket的返回值
http://blog.csdn.net/eroswang/archive/2010/06/02/5642550.aspx

C++的curl库调用手册
http://blog.csdn.net/eroswang/archive/2010/08/23/5832481.aspx

qsort和bsearch（qsort已经用过，bsearch是二分查找）
http://blog.csdn.net/eroswang/archive/2009/04/15/4075611.aspx

--------------------------------------------
协议
protobuffer(号称比json好)
http://blog.csdn.net/eroswang/archive/2010/11/16/6011566.aspx

--------------------------------------------
调试
显示16进制数据(vim和hexdump)
http://blog.csdn.net/eroswang/archive/2009/11/09/4791875.aspx

GCC内部都做些什么
http://blog.csdn.net/eroswang/archive/2010/11/03/5983791.aspx

杀掉僵尸进程
http://blog.csdn.net/eroswang/archive/2009/07/28/4388837.aspx

--------------------------------------------
内核等级
IOSTAT
http://blog.csdn.net/eroswang/archive/2009/04/24/4107322.aspx

malloc OS等级分析
http://blog.csdn.net/eroswang/archive/2009/04/27/4130972.aspx

