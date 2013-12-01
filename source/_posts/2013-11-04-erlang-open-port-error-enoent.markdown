---
layout: post
title: "erlang open_port 错误之error enoent"
date: 2013-11-04 11:04
comments: true
categories: [erlang,rrdtool] 
---
今天在另外一台机器上部署新写的服务端，运行后程序崩溃，最后发现erlang的find_executable的没有找到程序。

###问题重现：
<!-- more -->
在open_port处，查看日志报错如下：
```erlang
[rrddir ok]^
[ResRrd:{error,{enoent,[{erlang,open_port,
                                [{spawn_executable,"/usr/local/bin/rrdtool"},^
                                 [{line,1024},{args,["-"]}]],^
                                []},^M
                        {rrdtool,init,1,[{file,"src/rrdtool.erl"},{line,83}]},
                        {gen_server,init_it,6,^M
                                    [{file,"gen_server.erl"},{line,304}]},
                        {proc_lib,init_p_do_apply,3,
                                  [{file,"proc_lib.erl"},{line,227}]}]}}]
```
怀疑rrdtool-erlang这个库存在bug，移到rrdtool.erl代码处调查问题
{% codeblock  rrdtool.erl lang:erlang start:81 %}
%% @hidden
init([RRDTool]) ->
    Port = open_port({spawn_executable, RRDTool}, [{line, 1024}, {args, ["-"]}]),
    {ok, Port}.
{% endcodeblock %}

再定位到上层看gen_server的start函数
{% codeblock  rrdtool.erl lang:erlang start:55 %}
start() ->
    gen_server:start(?MODULE, [os:find_executable("rrdtool")], []).
{% endcodeblock %}

os:find_executable("rrdtool"),这个是用直接找到rrdtool可执行程序的调用，回到系统了输入rrdtool返回
```bash
RRDtool 1.4.7  Copyright 1997-2012 by Tobias Oetiker <tobi@oetiker.ch>
               Compiled Oct 22 2012 11:33:26

Usage: rrdtool [options] command command_options
Valid commands: create, update, updatev, graph, graphv,  dump, restore,
                last, lastupdate, first, info, fetch, tune,
                resize, xport, flushcached

RRDtool is distributed under the Terms of the GNU General
Public License Version 2. (www.gnu.org/copyleft/gpl.html)

For more information read the RRD manpages
```
已经装好了啊？那为什么还要报错，难道是做了alias
```bash
[yin@yin-arch monitorserver2]>alias rrdtool
/usr/lib64/rrdtool/bin/rrdtool
```
果不其然


###解决方法
在/usr/local/bin/目录下做一个软连接
```bash
sudo ln -s /usr/lib64/rrdtool/bin/rrdtool /usr/local/bin/rrdtool
```
再次运行，enoent问题解决。


