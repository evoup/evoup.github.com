---
layout: post
title: "关于log4erl的localtime_to_universaltime报错"
date: 2013-07-29 09:46
comments: true
categories:        erlang
---

关于logerl报错之

```bash
** {appender_died,

       {'EXIT',

           {badarg,

               [{erlang,localtime_to_universaltime,

                    [{ {2013,6,19},{15,52,35} },true],

                    []},
```
 

 
```bash
Eshell V5.9.1  (abort with ^G)

1> erlang:localtime_to_universaltime(DateTime, true).

* 1: variable 'DateTime' is unbound

2> erlang:localtime_to_universaltime(DateTime, true).

* 1: variable 'DateTime' is unbound

3> DateTime = { {2008,5,5},{1,1,1} }.

{ {2008,5,5},{1,1,1} }

4> erlang:localtime_to_universaltime(DateTime, true).

** exception error: bad argument

     in function  erlang:localtime_to_universaltime/2

        called as erlang:localtime_to_universaltime({ {2008,5,5},{1,1,1} },true)

5>
```
 

看来是个log4erl在freebsd下的bug，原因可能是作者没有考虑到下面的状况：
http://erlang.org/pipermail/erlang-bugs/2008-May/000787.html


后记：我的做法是把时区从UTC调整到CST，之后居然就一切OK了，关于log4erl如何调整时区到本地时间，查看手册https://github.com/ahmednawras/log4erl/blob/master/API.txt

```bash
 I - ISO format with universal GMT time (equivilant to "%jT%TZ").
 S - ISO format with local time and time zone offset
```
 也就是在配置中改成%S就可以了。
 
     