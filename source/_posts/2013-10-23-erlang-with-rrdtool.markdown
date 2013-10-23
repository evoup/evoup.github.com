---
layout: post
title: "如何在erlang中使用rrdtool进行监控数据的保存"
date: 2013-10-23 15:12
comments: true
categories:          [erlang,rrdtool,monitor]
---


##概述

项目需要保存监控数据，之前用hbase存然后再出图的方式，虽然数据量可以，但整个方式比较落后。

rrdtool是专门为了保存和出图设计的数据库。它的全称为round robin database，我们通常叫它为环状数据库。


关于如何创建rrd数据库的文章可以看这里http://www.cuddletech.com/articles/rrd/ar01s02.html


##创建RRD数据库

然后我们参考下开源监控软件ganglia的load_one数据库结构：


```
rrdtool info load_one.rrd

filename = "load_one.rrd"
rrd_version = "0003"
step = 15
last_update = 1382507991
ds[sum].type = "GAUGE"
ds[sum].minimal_heartbeat = 120
ds[sum].min = NaN
ds[sum].max = NaN
ds[sum].last_ds = "0.10"
ds[sum].value = 6.0000000000e-01
ds[sum].unknown_sec = 0
rra[0].cf = "AVERAGE"
rra[0].rows = 5856
rra[0].pdp_per_row = 1
rra[0].xff = 5.0000000000e-01
rra[0].cdp_prep[0].value = NaN
rra[0].cdp_prep[0].unknown_datapoints = 0
rra[1].cf = "AVERAGE"
rra[1].rows = 20160
rra[1].pdp_per_row = 4
rra[1].xff = 5.0000000000e-01
rra[1].cdp_prep[0].value = 3.3266666667e-01
rra[1].cdp_prep[0].unknown_datapoints = 0
rra[2].cf = "AVERAGE"
rra[2].rows = 52704
rra[2].pdp_per_row = 40
rra[2].xff = 5.0000000000e-01
rra[2].cdp_prep[0].value = 2.2742000000e+01
rra[2].cdp_prep[0].unknown_datapoints = 14
```

熟悉一下它的结构，数据库的名字叫做load_one.rrd，rrd的版本为3，步长step为15秒，即15秒之内的数据不能再次被写入，为一个最小单位。
然后last_update为最后一次更新的时间戳，数据类型为GAUGE，这是一种直接写入不做平均计算的数据类型。minimal_heartbeat为120秒，意思是120秒内没有数据被更新，系统认为状态未知。
min max为最大和最小。last_ds最后的ds为0.10，最后被写入的数据为6.0000000000e-01，就是0.6，未知的时间为0。
接下来CF的第一个AVERAGE的每行（row）有1个pdp（ Primary Data Point），共有5856个pdp,我们算下代表的时间跨度，15*1*5856=87840秒，为24.4小时。为啥有0.4小时，估计是出图的时候，为了更好看吧，可以不去管它。这里废话一句：也可以通过如下命令查看实际的时间跨度：
rrdtool dump load_one.rrd > load_one.xml
进去可以看一下是不是时间跨度规划正确。
![Alt text](/images/evoup/rrdtool_dump.png)

于是我有了我的load数据库

{% codeblock  foo.erl %}
{ok,PidRrdtool}=rrdtool:start(),
rrdtool:create(PidRrdtool, "load.rrd", [{"load", 'GAUGE', [120, 0, 100]}],
    [{'AVERAGE', 0.5, 1, 5856}, {'AVERAGE', 0.5, 4, 20160}, {'AVERAGE', 0.5, 40, 52704}
    ],[{step,15}]).
{% endcodeblock %}

 需要注意的是这个create会无条件重建数据库，所以每次运行要先判断是否存在，如果不存在
 才调用rrdtool:create函数创建数据库。
 
 其中最后一个参数为创建选项，可以传{step,15}，代表创建步长为15的数据库。
 
 
##更新数据库
这个比较简单了，就是update
```erlang
%%写入rrd数据库
%%Load为客户端上传的监控到的load数值
rrdtool:update(PidRrdtool, "load.rrd", [{"load", list_to_float(Load)}], now()).
```

##简单的绘图

这里用最原始的方法，rrdtool graph来画图
{% codeblock  make_graph.sh %}
#!/bin/sh
rrdtool graph  myLoad.png                    \
      --start 1382508875 --end 1382512874         \
      --title "Load Average yin-arch_ac101eb8"   \
      --v "Load Average"                          \
      DEF:load=load.rrd:load:AVERAGE              \
      HRULE:1#ff0000:"warning value"             \
      AREA:load#4A4A4A:load\ average\
{% endcodeblock %}

运行该脚本，最后绘图效果见此:

![Alt text](/images/evoup/rrdtool_load_graph.png)
 
 
 
 
其他参考资料：
http://oss.oetiker.ch/rrdtool/

https://github.com/Vagabond/erlang-rrdtool

http://blog.sina.com.cn/s/blog_79d1f5e00100test.html

