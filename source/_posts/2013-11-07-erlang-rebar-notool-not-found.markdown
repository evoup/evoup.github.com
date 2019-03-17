---
layout: post
title: "rebar生成发布包后找不到noodtool"
date: 2013-11-07 15:08
comments: true
categories: erlang 
---
今天用rebar编译出目标文件准备直接部署到目标服务器上，结果报告

{% codeblock lang:bash %}
[evoup@host63i386 erlang]>sudo rel/madmonitor2/bin/madmonitor2 start
current node: madmonitor2@192.168.216.165
release vm.args file empty,recopy
escript: Failed to open file: /usr/home/evoup/project/madmonitor2/erlang/rel/madmonitor2/erts-

5.9.3.1/bin/nodetool
{% endcodeblock %}

<!-- more -->

原来是少一个nodetool文件，进到目录一看发现有2个erts
一个是5.9.1，一个是5.9.3.1，分别代表R15B01和R15B03，想起之前系统装过2个版本的erlang，R15B03的没有删除干净就装了R15B01,看来是rebar自作主张读取了2个erlang运行时库。
删除之前的剩余文件

{% codeblock lang:bash %}
sudo rm -rf /usr/local/lib/erlang/erts-5.9.3.1/
sudo rm -rf /usr/local/lib/erlang/lib/erts-5.9.3.1/
{% endcodeblock %}

仔细一看还有其他版本的erlang，rebar还没有这么傻:)
都给删除了

{% codeblock lang:bash %}
sudo rm -rf /usr/local/lib/erlang/erts-5.5.1/
sudo rm -rf /usr/local/lib/erlang/erts-5.6.5/
{% endcodeblock %}

再次编译，问题解决。
