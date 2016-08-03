---
layout: post
title: "【How to】使用basho的erlang-protobuffs和PHP通讯"
date: 2013-05-27 13:49
comments: true
categories:     [erlang,php]
---
这几天通过查阅相关资料，终于实现了php和erlang的相互通讯。详细的protocolbuf的入门，可以参见本问最后的参考链接。有文章说google官网实现的protobuf的erlang接口不是很好用，推荐使用basho的erlang-protobuffs修改版本。于是摸索了一番，终于勉强可以使用了，现先介绍如何使用该工具生成消息的erlang源文件。米聊用的分布式数据库riak就是basho公司开发的，其中采用了修改版erlang-protobuffs。

<!-- more -->

可以在这里下载，git://github.com/basho/erlang_protobuffs.git

先安装，我基本没有很好的安装，直接放到临时路径，要生成源代码的时候，进入该目录，生成，然后复制生成好的源代码文件到自己的项目目录里。

```bash
cd erlang_protobuffs
make all
```

这样就可以使用了，接下来给出一个测试的protobuf文件

{% codeblock lang:protobuf test.proto %}
message test {
   required string test1 = 1;
}
{% endcodeblock %}


进入ebin目录准备生成

```bash
cd ebin  
```
           
```erlang
erl
1> protobuffs_compile:generate_source("test.proto").

=INFO REPORT==== 27-May-2013::15:31:13 ===

Writing header file to "test_pb.hrl"

=INFO REPORT==== 27-May-2013::15:31:13 ===

Writing src file to "test_pb.erl"

ok
```

这样生成就完毕了，一共生成2个文件test_pb.hrl和test_pb.erl。

然后写一个server端测试，照搬erlang程序设计中的最入门的单线程例子改了改，一处理完就断的那种,在代码里引入该文件。

{% codeblock lang:erlang server.erl start:51 mark:51,54-55 %}
-module(server).

-compile(export_all).

-import(lists, [reverse/1]).

-include("test_pb.hrl").

start_nano_server() ->

   {ok, Listen} = gen_tcp:listen(2345, [binary, {packet, 4},

                    {reuseaddr, true},

                    {active, true}]),

   {ok, Socket} = gen_tcp:accept(Listen),

   gen_tcp:close(Listen),

   loop(Socket).


loop(Socket) ->

   receive

   {tcp, Socket, Bin} ->

       io:format("Server received binary = ~p~n",[Bin]),

       Msg=test_pb:decode_test(Bin),

       io:format("Server (unpacked)  ~p~n",[Msg]),

       loop(Socket);

   {tcp_closed, Socket} ->

       io:format("Server socket closed~n")

   end.
{% endcodeblock %}


client的代码

{% codeblock lang:erlang client.erl %}
-module(client).

-compile(export_all).

-import(lists, [reverse/1]).

-include("test_pb.hrl").

nano_client_eval() ->

   {ok, Socket} =

   gen_tcp:connect("localhost", 2345,

       [binary, {packet, 4}]),

   Test=#test{test1="test1"},

   Str=test_pb:encode_test(Test),

   ok = gen_tcp:send(Socket, Str),

   gen_tcp:close(Socket).
{% endcodeblock %}

需要说明的是packet,4这个参数，代表每个数据包的前4个字节为消息头，该头标识了消息体的长度。这样互通是没有问题的，erlang自动为数据包的加上前4个字节的消息头。

接下来难点是php作为客户端，要手工打包消息然后发送。 
首先是下载php版本的https://code.google.com/p/pb4php/
然后，解压得到protocolbuf，接着创建你的项目，把protocolbuf放到该项目文件夹的根目录下。一样要生成源代码。注意，pb4php不是很智能，如果直接.proto文件中=1没有空格，而不是写成xx = 1是会报错的！

{% codeblock lang:php test.php %}    
<?php

require_once("./protocolbuf/parser/pb_parser.php");

$parser = new PBParser();

$parser->parse("./test.proto");

echo "done\n";

?>
{% endcodeblock %}

运行test.php后生成pb_proto_test.php

php版本的client的代码

{% codeblock lang:php client.php %}
<?php

require_once("./protocolbuf/message/pb_message.php");

require_once("./pb_proto_test.php");

$test = new test();

$test->set_test1("test php");

$string = $test->SerializeToString();

$sock=@socket_create(AF_INET, SOCK_STREAM, getprotobyname('tcp'));


if ($sock)
{

   socket_connect($sock, "127.0.0.1", 2345);

   $msg = pack_data($string);

   file_put_contents(dirname(__FILE__).'/resmessage',$msg);

   $sent = @socket_write($sock, $msg, strlen($msg));
}

function pack_data ($data) {

   $head =pack("H*", to_hex_str (strlen($data)));

   $body=pack("A*",$data);

   return $head.$body;

}


function to_hex_str ($num)
{

   $str = dechex($num);

   $str = str_pad($str,8,"0",STR_PAD_LEFT);

   return $str;

}
?>
{% endcodeblock %}

运行上面的php客户端可以和erlang版本的server.erl服务端实现二进制CS互通。期间由于不理解erlang的packet含义，用抓包查了一下才搞定的。不明白原理的，可以尝试抓一下包，然后测试。同时获取了消息体的长度后，可用php的函数dechex()函数获取十六进制代码，然后有这样一个规律。如果erlang服务端packet参数后为2，则str_pad($str,4,"0",STR_PAD_LEFT)然后打包消息头，如果服务端packet参数后跟4，则str_pad($str,8,"0",STR_PAD_LEFT)然后打包消息头。

最后，还有一个要注意，erlang版本的protocolbuf不知道为什么，int32和int64最多不能超过10位，在项目里我一概成了string类型。其他，optinal类型对于不一定出现的数据也是很好用的。

