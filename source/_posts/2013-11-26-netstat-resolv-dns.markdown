---
layout: post
title: "部分机器netstat运行结果缓慢的原因"
date: 2013-11-26 14:17
comments: true
categories: shell 
---

今天看了下客户端运行状况，发现流量图断断续续的。直觉反应到可能是自己按照netstat源码修改的erlang的NIF扩展存在BUG，会奔溃。但查了下似乎没有这个问题。

![Alt text](/images/evoup/netstat1.png)
<!-- more -->

该机器执行netstat本身就很慢，top看了下CPU占用也无异常，该机器基本OK，就是netstat执行起来很慢。
后来发现使用-n参数就快了。
```
     -n    Show network addresses and ports as numbers.  Normally netstat
           attempts to resolve addresses and ports, and display them symboli‐
           cally.
```
这个参数是说netstat以数字的方式展现网络地址和端口。而不加该参数时netstat会尝试去解析地址和端口，然后采用象征方式呈现。至于究竟啥叫象征方式，其实就是如果你的hostname是localhost,它就会显示localhost，而不是127.0.0.1的方式。
最后判断应该是本机的出站通讯没有加上dns协议的53端口导致的，经过参数修改，netstat调用正常了。
