---
layout: post
title: "关于socket阻塞与非阻塞情况下的recv、send、read、write返回值"
date: 2013-01-09 11:42
comments: true
categories: [socket server]
---
####recv：

阻塞与非阻塞recv返回值没有区分，都是 <0：出错，=0：连接关闭，>0接收到数据大小，

特别：非阻塞模式下返回 值 <0时并且(errno == EINTR || errno == EWOULDBLOCK || errno == EAGAIN)的情况 下认为连接是正常的，继续接收。

<!--more -->

只是阻塞模式下recv会阻塞着接收数据，非阻塞模式下如果没有数据会返回，不会阻塞着读，因此需要 循环读取。


####write：

阻塞与非阻塞write返回值没有区分，都是 <0：出错，=0：连接关闭，>0发送数据大小，

特别：非阻塞模式下返回值 <0时并且 (errno == EINTR || errno == EWOULDBLOCK || errno == EAGAIN)的情况下认为连接是正常的， 继续发送。

只是阻塞模式下write会阻塞着发送数据，非阻塞模式下如果暂时无法发送数据会返回，不会阻塞着 write，因此需要循环发送。

 
####read：

阻塞与非阻塞read返回值没有区分，都是 <0：出错，=0：连接关闭，>0接收到数据大小，

特别：非阻塞模式下返回 值 <0时并且(errno == EINTR || errno == EWOULDBLOCK || errno == EAGAIN)的情况 下认为连接是正常的，继续接收。

只是阻塞模式下read会阻塞着接收数据，非阻塞模式下如果没有数据会返回，不会阻塞着读，因此需要 循环读取。


####send：

阻塞与非阻塞send返回值没有区分，都是 <0：出错，=0：连接关闭，>0发送数据大小，

特别：非阻塞模式下返回值 <0时并且 (errno == EINTR || errno == EWOULDBLOCK || errno == EAGAIN)的情况下认为连接是正常的， 继续发送。

只是阻塞模式下send会阻塞着发送数据，非阻塞模式下如果暂时无法发送数据会返回，不会阻塞着 send，因此需要循环发送。
