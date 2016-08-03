---
layout: post
title: "mochiweb下调用rrdtool graph命令绘制类似ganglia的load的图表"
date: 2013-10-28 16:41
comments: true
categories:       [erlang,rrdtool]
---

今天做rrdtool图表，发现ganglia的图表还是不错的，我就一个一个来模仿，先从one minute load开始。通过mochiweb在restful接口直接出图。
<!-- more -->

```erlang
-module(webapi_func_graph).
-include("include/inc.hrl").
-compile(export_all).

get(Selector,Req,Key) ->
    Method=Req:get(method),
    QueryStringData = Req:parse_qs(),
    io:format("[req:~p][qsd:~p]~n",[Req,QueryStringData]),
    StartTs = proplists:get_value("start", QueryStringData, ""),
    EndTs = proplists:get_value("end", QueryStringData, ""),
    Width = proplists:get_value("w", QueryStringData, "705"),
    Height = proplists:get_value("h", QueryStringData, "245"),
    io:format("[M:~p][F:get][A:~p][M:~p][K:~p][start:~p][end:~p][width:~p][height:~p]~n",[?MODULE,Selector,Method,Key,StartTs,EndTs,Width,Height]),
    case Selector of
        "@self" ->
            FileName="/tmp/myLoad_"++integer_to_list(erlang:phash2(make_ref()))++".png",
            AvgLoad= os:cmd("/usr/local/bin/rrdtool fetch /services/rrds/"++Key++"/load.rrd AVERAGE --start "++StartTs++" --end "++
                EndTs++" | awk 'BEGIN {ORS=\"\"} {sum+=$2} END {print sum/NR}'"),
            AverageLoad=list_to_float(AvgLoad),
            io:format("[average load:~p]~n",[AverageLoad]),
            case AverageLoad > 2 of
                true ->
                    case AverageLoad > 10 of
                        true ->
                            BColor="FF9966";
                        false ->
                            BColor="E2ECFF"
                    end;
                _ ->
                    BColor="CCFF99"
            end,
            os:cmd("/usr/local/bin/rrdtool graph "++FileName++" --lazy --start "++StartTs++" --end "++EndTs++"  --title \"One Minute Load Average "++Key++" last hour\" --width "++Width++" --height "++Height++" DEF:load=/services/rrds/"++Key++"/load.rrd:load:AVERAGE AREA:load#4A4A4A:load GPRINT:load:LAST:\" Current\\:%8.2lf %s\"  GPRINT:load:AVERAGE:\"Average\\:%8.2lf %s\\n\"  GPRINT:load:MAX:\"Maximum\\:%8.2lf %s\" GPRINT:load:MIN:\"Minimum\\:%8.2lf %s\" -c BACK#"++BColor),
            {ok, Data} = file:read_file(FileName),
            file:delete(FileName),
            {"Content-type: image/png",Data};
        _ ->
            io:format("[other]~n")
    end.
```

注意点，我没有使用rrdcgi这个东西是因为，手册上没有说可以直接输出二进制文件流，第二即使可以用，mochiweb也没有cgi的接口可以对接。
访问我的restful接口http://192.168.216.145/mmsapi2.0beta/get/graph/@self/yin-arch_ac101eb8?start=1382927568&end=1382944644&w=395&h=141

最后出图像下面一样的图了。当平均load大于10背景呈现红色（代表严重），大于2为蓝色，其余绿色。计算平均load我使用了rrdtoo fetch这个指令，此外还要注意在/tmp目录生成好随机名图片，用完了之后得删除，这个也是惯用方法。其余的请参阅rrdtool graph和rrdtool fetch命令的说明。
![Alt text](/images/evoup/yin-arch_ac101eb8_load.png)

参考链接：
http://man.lupaworld.com/content/manage/ringkee/awk.htm



