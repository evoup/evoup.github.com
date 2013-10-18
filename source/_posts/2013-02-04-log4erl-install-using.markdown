---
layout: post
title: "log4erl 安装使用"
date: 2013-02-04 16:57
comments: true
categories:   erlang 
---    

目前项目急需一套可记录到文件并能自动轮询的机制的日志系统，考虑良久，最后置自带gen_event和标准日志函数于不顾，决定用log4erl。

支持多日志

当前文件Appender仅支持基于大小的日志文件滚动

支持默认Logger,未指定Logger时系统提供默认Logger

5个预定义的日志级别(debug, info, warn, error, fatal)

一个error_logger的日志处理器

支持用户指定日志级别

支持日志格式化

支持控制台日志

支持smtp formatter

支持XML格式的日志

支持syslog

支持在运行时改变Appender的格式和级别.

下载地址：https://github.com/ahmednawras/log4erl

官方手册：http://code.google.com/p/log4erl/wiki/Log4erl_Manual_2

安装log4erl

```bash
cd log4erl
make
```
安装的过程中会要求mochiweb的库文件也被安装 Dependency not available: mochiweb-1.5.1 

只需要先make depends就可以了。

```bash
cd log4erl
erl
> make:all([{outdir,"../ebin"}]).
```
然后安装到erlang的库文件中


```bash
cp -Rf log4erl /usr/local/lib/erlang/lib/log4erl
```
进入erl应用目录后

```bash
erl
> application:start(log4erl).
```

然后创建好应用的log配置文件

```erlang
> log4erl:conf("priv/log4erl.conf").
```

这里说一句，这是在log4erl的目录，如果是自己的应用，到priv目录下创建log4erl.conf就可以了。log4j默认提供了2个配置文件，可以根据需要自己选择一个进行修改，文章最后提供了这2个文件。

或者可以使用编程方式来动态配置

```erlang
> log4erl:add_logger(messages_log).
> log4erl:add_console_appender(messages_log, cmd_logs, {warn, "[%L] %l%n"}).
```

API的使用

```erlang
> log4erl:info("Information message").
```

日志的等级

all = debug  < info < warn < error < fatal < none 

shell下的测试

```erlang
erl
>application:start(log4erl).
>log4erl_conf:conf("/path/to/conf/file").
>log4erl:log(warn,"log something").
```
<hr>
最后提供1个默认配置文件，可以结合官方文档进行设置。一般log4erl_conf:conf只接收其中之一的文件路径。

```erlang
%% Default logger
%% it includes a file appender and a console appender
logger{
        file_appender app2{
                dir = "/tmp/",
                level = info,
                file = madmonitor2,
                type = time,
                max = 600,
                suffix = log,
                rotation = 4,
                format = '[%L]: %S, %l%n'
        }

        console_appender app1{
                level = warn,
                format = '%T %j [%L] %l%n'
        }
}
```

其他注意：使用记录到文件的方式，不支持中文！事先最好全部转换到英文格式。
