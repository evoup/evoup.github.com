---
layout: post
title: "erlang版的long2ip函数"
date: 2013-10-09 17:56
comments: true
categories:    octopress 
---
工作中参考网络文章整理了一个整形转IP地址的函数。直接看代码了。


{% codeblock erlang代码片段 lang:erlang %}
%%从整形转换为IP地址元组
long2ip(IpInteger) ->
    Integer_to_ip=fun(Ip)-> {Ip bsr 24, (Ip band 16711680) bsr 16, 
        (Ip band 65280) bsr 8, Ip band 255} end,
    Integer_to_ip(IpInteger).
{% endcodeblock %}

这样子使用
long2ip（3232290954).
{192,168,216,138}