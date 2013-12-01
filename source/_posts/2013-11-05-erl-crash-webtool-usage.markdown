---
layout: post
title: "使用webtool查看erl_crash.dump"
date: 2013-11-05 14:17
comments: true
categories: erlang 
---

运行erlang程序崩溃，按照许多人推荐的webtool方式找问题，这里记录一下启动的过程。
<!-- more -->

首先开一个erl shell，然后按如下参数启动webtool
webtool:start(standard_path,[{port,8888},{bind_address,{192,168,216,145}},{server_name,"monitorserver2"}] ).

然后访问浏览器可以调试到erl.crash的具体信息。


![Alt text](/images/evoup/webtool_watch__erlcrash.png)

最后发现虽然没有找到什么有价值的信息，总算不是竹篮打水，最后通过记录日志的方式解决了问题。在这里先记录一下了……

参考链接：

http://www.erlang.org/doc/man/webtool.html#start-2
http://hi.baidu.com/ah__fu/item/86c65e8b88ecf453e73d1962


