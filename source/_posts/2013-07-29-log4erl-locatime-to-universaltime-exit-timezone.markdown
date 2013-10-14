---
layout: post
title: "关于log4erl的localtime_to_universaltime报错"
date: 2013-07-29 09:46
comments: true
categories:        erlang
---

##问题症状：
项目遇到编译后运行的一个问题，logerl报错之

```erlang
** {appender_died,

       {'EXIT',

           {badarg,

               [{erlang,localtime_to_universaltime,

                    [{ {2013,6,19},{15,52,35} },true],

                    []},
```
看来是无法进行本地时间和UTC的转换，继续尝试一下这个localtime_to_universaltime函数
 
<!-- more -->
 
```erlang
Eshell V5.9.1  (abort with ^G)

1> DateTime = { {2008,5,5},{1,1,1} }.

{ {2008,5,5},{1,1,1} }

2> erlang:localtime_to_universaltime(DateTime, true).

** exception error: bad argument

     in function  erlang:localtime_to_universaltime/2

        called as erlang:localtime_to_universaltime({ {2008,5,5},{1,1,1} },true)

5>
```
 

看来是个log4erl遭遇了在freebsd下调用localtime_to_universaltime函数的bug，原因可能是作者没有考虑到下面的状况：
<a href="http://erlang.org/pipermail/erlang-bugs/2008-May/000787.html" target=_BLANK>http://erlang.org/pipermail/erlang-bugs/2008-May/000787.html</a>

##解决方法：
我的做法是使用sysinstall指令把时区从UTC调整到CST，之后运行居然就一切OK了。

##时区调整：
关于log4erl如何调整时区到本地时间，查看手册<a href="https://github.com/ahmednawras/log4erl/blob/master/API.txt" target=_BLANK>https://github.com/ahmednawras/log4erl/blob/master/API.txt</a>

```bash
 I - ISO format with universal GMT time (equivilant to "%jT%TZ").
 S - ISO format with local time and time zone offset
```
 也就是在配置中改成%S就可以了。
 
     