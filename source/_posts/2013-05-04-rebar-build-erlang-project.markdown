---
layout: post
title: "[How to]使用rebar构建erlang 项目"
date: 2013-05-04 14:41
comments: true
categories:    erlang 
---

rebar这个程序其实是个escript脚本，通过它可以对erlang项目创建、编译、生成和升级。之前一直使用着《Erlang/OTP并发编程实战》EMakefile的方法，进行项目的编译，也很简单，但是后续的又有一套发布的机制。现在很多项目在用rebar，于是我也试了一下，没想到问题还蛮多的，在这里记一下，我只是用了绕过的方法不是很好，因为时间不允许，这里先SORRY一下。我这里的环境为freebsd9.0 amd64 (elrang5.9.1 R15B01),erlang的其他版本，请参看wiki的Release-handling部分调整生成环节的命令。

1）第一步要获取rebar程序，两种方法：自己编译和直接下载。

<!-- more -->

自己编译：
https://github.com/rebar/rebar
直接按照README.md的方法编译出一个rebar。

直接下载：

{% codeblock lang:bash %}
curl -o rebar http://cloud.github.com/downloads/basho/rebar/rebar
{% endcodeblock %}

这个就直接可用了。

无论哪种方式搞定之后，进入自己项目的根目录都要chmod +x rebar给它可执行权限。

2）创建项目

在项目的根目录
{% codeblock lang:bash %}
~/exemplar $ mkdir -p apps/exemplar

~/exemplar $ cd apps/exemplar

~/exemplar $ rebar create-app appid=exemplar
{% endcodeblock %}

这样就在项目根目录下得到了app文件夹，该文件夹下的src文件夹中默认有三个项目源文件，一个src文件，一个主文件，一个监督者文件。接下来就像往常一样的编写erlang代码。（甚至我当初直接在该目录用Emakefile的方式进行调试也是可以的，只是路径稍有变化，这里只带过）

3）生成发布版
{% codeblock lang:bash %}
~/exemplar $ mkdir rel

~/exemplar $ cd rel
{% endcodeblock %}

接下来非常怂的问题来了，编辑reltool.config时wiki上说把
{% codeblock lang:erlang %}
{sys, [

       {lib_dirs, []},

{sys, [

    {lib_dirs, ["../apps"]},
{% endcodeblock %}

其实还需要加上依赖的路径
{% codeblock lang:erlang %}
    {lib_dirs, ["../apps", "../deps"]},
{% endcodeblock %}

这样才对。

然后还有在  {rel, "exemplar", "1",

后面一段，生成发布的时候还会提醒要求带上sasl、inet、crypto
而在 {excl_app_filters, ["\.gitignore"]},后面
依次相应要带上
{% codeblock lang:erlang %}
       {app, sasl,   [{incl_cond, include}]},
       {app, mochiweb,   [{incl_cond, include}]},
       {app, inets,   [{incl_cond, include}]},
       {app, crypto,   [{incl_cond, include}]},
{% endcodeblock %}

到这里reltool.config的麻烦才告一段落。
最后回到根目录
rebar.config的内容如下
{% codeblock lang:erlang %}
{deps, [
    {mochiweb, "1.5.1",
        {git, "git://github.com/mochi/mochiweb.git",
            {tag, "1.5.1"}}},
        {'log4erl', ".*",
            {git, "git://github.com/ahmednawras/log4erl.git",
                "master"}}
{sub_dirs, ["apps/exemplar", "rel"]}.
{% endcodeblock %}

好了，rebar.config，我这里也不多说，反正我是试了各种版本，要么生成的so变成了exemplar_drv.so，要么干脆就无法生成。我最后放到Makefile里一并解决了。
下面是这个Makefile的内容，其中make.sh是自行生成erlang的NIF外围C/C++程序扩展so的编译脚本，这个要注意一下(我没有创建apps/c_src这个文件夹，而是在apps文件夹下又放了个files文件，里面是c的源文件，这样make.sh的内容就大概是gcc -Wall -o $APP_PREFIX/priv/nif.so -fpic -shared -I $ERL_LIB/erts-$ERL_VER/include/ apps/files/nif.c 这样子，反正还是要生成到priv路径就对了)
{% codeblock lang:makefile %}
cat Makefile
all: compile
depends:
        ./rebar get-deps
        ./rebar update-deps
compile:
        ./rebar compile
        ./make.sh
clean:
        ./rebar clean
release:
        ./rebar generate
        mkdir ./rel/exemplar/priv
        cp -r ./apps/exemplar/priv/* ./rel/exemplar/priv/
        chmod +x ./rel/exemplar/bin/exemplar
.PHONY: all depends compile
{% endcodeblock %}

做Makefile的方法，也是我参考了github上一些开源erlang项目的想到的办法。
所有这一切做完按照如下方式就可以生成发布了。
{% codeblock lang:makefile %}
    make clean
    make depends
{% endcodeblock %}

    编辑./deps/log4erl/src/log4erl.app.src文件，把{vsn, "0.9.0"}修改为{vsn, ""}，新版本的没试过，可能版本号不一致，请自行修改！
{% codeblock lang:makefile %}
    make release
{% endcodeblock %}

基本上要注意的就是这些，以下为参考链接：

[https://github.com/rebar/rebar/wiki](rebar wiki)

[https://github.com/rebar/rebar/wiki/Release-handling](rebar wiki release-handling)

[rebar工具使用备忘录 (1)](http://cryolite.iteye.com/blog/1159448)  rebar工具使用备忘录，这个比较早期so_specs已经被port_specs取代了，有需要的联系起来看看吧

[Rebar：Erlang构建工具](http://www.cnblogs.com/panfeng412/archive/2011/08/14/2137990.html)
