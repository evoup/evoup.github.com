---
layout: post
title: "rrdtool graph报错Cannot parse vname from DEF"
date: 2013-4-13 14:55
comments: true
categories:  rrdtool
---

今天在创建rrdtool的graph的时候遇到了报错Cannot parse vname from 'DEF

```sh
#/bin/sh
/usr/local/bin/rrdtool graph result.png --lazy --start 1384320009 --end 1384323609 --title "Disk / usage" -v "bytes" --width 320 --height 240 \
DEF:/=/services/rrds/yin2-monitorbeta-arch_ac101eb4/diskused_.rrd:/:AVERAGE AREA:/#444444
```

<!-- more -->

最后发现其实是保存的时候vname名字是used，非常白痴的低级错误记录一下,贴出正确的统计/挂载点的used指标图表呈现代码。

```sh
#/bin/sh
/usr/local/bin/rrdtool graph result.png --lazy --start 1384320009 --end 1384323609 --title "Disk / usage" -v "bytes" --width 320 --height 240 \
DEF:used=/services/rrds/yin2-monitorbeta-arch_ac101eb4/diskused_.rrd:used:AVERAGE AREA:used#444444
```



