---
layout: post
title: "erlang自带日志写文件的方法"
date: 2013-02-04 18:19
comments: true
categories:     erlang
---

最近读erlang/OTP并发编程介绍第七章的日志，发现居然没有记录到日志的方法。这种方式打印的日志，仅仅是输出到了控制台。自带SASL日志有很多局限，比如没有内置记录到文件的功能。其实还可以考虑一下log4erl，这里先介绍下自带日志的写文件的方法。

```erlang
-module(x).

-define(LogsLoaction,"/tmp/").

-export([start/0]).

start()->

   FileName = ?LogsLoaction++"log.txt",

   file:write_file(FileName, <<"start to record">>),

   error_logger:logfile({open,FileName}),

   info("[init] log start ~n").

info(String)-> error_logger:info_msg(String).

info(String,Value)-> error_logger:info_msg(String,Value).

error(String)-> error_logger:error_msg(String).

error(String,Value)-> error_logger:error_msg(String,Value).
```

对于日志文件的轮询还要使用newsyslog等软件进行轮询。本文相对开源的erl日志系统log4erl还是属于比较山寨的解决方案，没有进行系统的测试，仅能使用。:)。
        
        