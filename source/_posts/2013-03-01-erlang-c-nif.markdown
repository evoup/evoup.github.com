---
layout: post
title: "erlang调用C库函数(NIF 方式)"
date: 2013-03-01 13:39
comments: true
categories:         erlang
---
新客户端要求节省系统开销，对于监控的项目，如cpu的load不要通过shell的方式（如top）来获取，通过阅读先关开源软件的代码发现了getloadavg这个C标准库函数。考虑erlang三种和C外围程序交互的方式中，如果开发同步程序的话，NIF（本地函数）是在最高效的。但也要注意一下不要把费事阻塞的操作写到NIF中去，否则会vm会一直等待。

好久没写C的动态库了，记得上次还是学习vc的时候去调用dll,还好unix下做动态库很容易。这就开始，拿到《erlang OTP并发编程》翻到十二章，发现该书OTP的版本有R13和R14。我手头的版本是R15，

<!-- more -->

参考了例子和这篇文章《Erlang NIF简析》http://blog.csdn.net/keyeagle/article/details/7009208

终于实验成功。

{% codeblock cgetloadavg.c lang:c %}
#include "erl_nif.h"
#include <stdlib.h>
#include <stdio.h>

static ERL_NIF_TERM getloadavg_ex(ErlNifEnv* env, int argc, const ERL_NIF_TERM argv[])
{
   double load[3];
   char buf[30];
   if (getloadavg(load,3)==-1) {
       sprintf(buf, "%f", 0);
   } else {
       sprintf(buf, "%f", load[0]);
   }
   return enif_make_string(env, buf, ERL_NIF_LATIN1);
}

static ErlNifFunc nif_funcs[] =
{
   {"getloadavg_ex", 0, getloadavg_ex}
};

ERL_NIF_INIT(getloadtest,nif_funcs,NULL,NULL,NULL,NULL)
{% endcodeblock %}
以下写下简单的调用过程：

```bash
{% codeblock lang:bash %}
gcc -o getloadtest.so -fpic -shared -I /usr/local/lib/erlang/erts-5.9.1/include/ cgetloadavg.c
{% endcodeblock %}


还没有在app里试验，如果是标准app
{% codeblock lang:bash %}
gcc -o ./priv/getloadtest.so -fpic -shared -I /usr/local/lib/erlang/erts-5.9.1/include/ cgetloadavg.c
{% endcodeblock %}

就可以生成供NIF调用的动态链接库

然后是erlang的，调用很简单：


{% codeblock getloadtest.erl lang:erlang %}
-module(getloadtest).
-export([init/0, getloadavg_ex/0]).
-on_load(init/0).

init() ->
   erlang:load_nif("./getloadtest", 0).

getloadavg_ex() ->
   "NIF library not loaded".
{% endcodeblock %}

{% codeblock lang:erlang %}
Eshell V5.9.1  (abort with ^G)
1> c(getloadtest).
{ok,getloadtest}
2> getloadtest:getloadavg_ex().
"2.263184"
{% endcodeblock %}

reference:

[http://www.erlang.org/doc/tutorial/nif.html](http://www.erlang.org/doc/tutorial/nif.html)

[http://www.erlang.org/doc/man/erl_nif.html](http://www.erlang.org/doc/man/erl_nif.html)

[http://blog.csdn.net/d52787790/article/details/7103288](http://blog.csdn.net/d52787790/article/details/7103288)

[http://blog.csdn.net/keyeagle/article/details/7009208](http://blog.csdn.net/keyeagle/article/details/7009208)

[http://www.freebsd.org/cgi/man.cgi?query=getloadavg](http://www.freebsd.org/cgi/man.cgi?query=getloadavg)

