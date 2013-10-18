---
layout: post
title: "[转]Erlang监测系统CPU、内存和磁盘"
date: 2013-02-27 13:59
comments: true
categories:     erlang
---
os_mon 
Erlang的os_mon服务中提供了一些用于监测系统信息的服务

<font color="red">(PS:经过测试，freebsd上使用get_disk_data无法获取到磁盘的数据)</font>

cpu_sup：监测CPU负载和使用率（Unix）

disksup：监测磁盘（Unix、Windows）

memsup：监测内存（Unix、Windows、VxWorks）

os_sup：监测系统日志（Solaris、Windows）

<!-- more -->

使用os_mon进行监测先必须启动监测服务application:start(os_mon) ，因为os_mon服务依赖于sasl服务，先必须启
动sasl服务，application:start(sasl) ，否则会返回{ error,{not_started,sasl} } 错误。os_mon提供的四种监测服
务中默认会启动三种服务：cpu_sup、disksup和memsup，如果需要自己设置启动的监测服务，可以修改os_mon.app
文件中的配置参数

start_cpu_sup = bool()

start_disksup = bool()

start_memsup = bool()

start_os_sup = bool()

等于true时启动，等于false时不启动。os_mon.app文件在erlang的安装目录下../erl5.8.3/lib/os_mon-2.2.5
/ebin 。

cpu_sup
 
cpu监测在5.8.3版本中只能用于Solaris和Linux操作系统，负载值与Unix进程运行前在队列中的排队时间成正比，
因此值越大意味着负载越高，返回值除以256为rup和top命令中显示的值。avg1/0,avg5/0 和avg15/0 函数计算负载，
util/0 和util/1 函数计算CPU使用率。在Linux系统中，必须保证/proc文件目录能被cpu_sup服务访问，如果不能监
测服务会停止

模块中的函数列表

nprocs() -> UnixProcesses | {error, Reason}

返回UNIX进程数

avg1() -> SystemLoad | {error, Reason}

返回最后1分钟系统的负载

avg5() -> SystemLoad | {error, Reason}

返回最后5分钟系统的负载

avg15() -> SystemLoad | {error, Reason}

返回最后15分钟系统的负载

util() -> CpuUtil | {error, Reason}

返回CPU使用率

util(Opts) -> UtilSpec | {error, Reason}

返回CPU使用率的详细信息

调用这些函数取CPU监测数据时，如果前后两次调用，数值没有变化时显示为0，有点奇怪

disksup
 
disksup是一个用来监测磁盘空间的进程，适用于Unix和Windows系统。监测服务定期检查磁盘，对于每个磁盘或分
区，在它使用超过一定的可用空间量，通过{ {disk_almost_full，MountedOn}，[] } 设置产生报警。在Unix下所有的
本地磁盘都会被监测，包括存在的交换分区。在WIN32下所有类型为“FIXED_DISK”逻辑驱动器都会被检查。
配置监控间隔时间和阀值

disk_space_check_interval = int()>0

监测间隔时间，单位为分钟，默认为30分钟。

disk_almost_full_threshold = float()

监测阀值，磁盘使用率达到多少时产生告警，默认为80，单位是百分比。

模块中的函数列表

get_disk_data() -> [DiskData]

返回最后一次磁盘检查结果

get_check_interval() -> MS

获取监测间隔时间，单位是毫秒

set_check_interval(Minutes) -> ok

设置监测间隔时间，这个设置在下一次监测时生效，服务退出后，这个值会失效，重启服务后使用默认值

get_almost_full_threshold() -> Percent

获取监测阀值，为磁盘使用率

set_almost_full_threshold(Float) -> ok

设置监测阀值，服务重启后，设置失效，使用默认值

memsup
 
memsup用来监控系统内存和各个进程内存的使用率，适用于Unix、Windows和VxWorks系统，定时监测内存，如果内
存使用超过系统分配的一定值，通过{system_memory_high_watermark, []} 设置产生告警。如果系统中任何Erlang
进程使用内存超过在总内存中的一定百分比，通过设置{process_memory_high_watermark,Pid} 产生告警。

配置监测间隔时间和阀值

memory_check_interval = int()>0

以分钟为刻度，默认为1分钟

system_memory_high_watermark = float()

内存使用阀值，默认为80，单位是百分比

process_memory_high_watermark = float()

单个Erlang进程使用阀值，默认为5，单位是百分比

memsup_helper_timeout = int()>0

等待监测结果的超时时间，默认为30秒

memsup_system_only = bool()

设置是否只监控系统内存使用率还是同时监测Erlang进程内存使用率，默认为false

模块中的函数列表

get_memory_data() -> {Total,Allocated,Worst}

获取系统总内存，使用内存，每个Erlang进程的使用内存

get_system_memory_data() -> MemDataList

获取系统内存使用的详细信息

get_os_wordsize() -> Wordsize

获取操作系统的位数

get_check_interval() -> MS

获取监测间隔时间，单位毫秒

set_check_interval(Minutes) -> ok

设置监测间隔时间，单位分钟

get_procmem_high_watermark() -> int()

获取每一进程内存使用告警阀值

set_procmem_high_watermark(Float) -> ok

设置每一进程内存告警阀值

get_sysmem_high_watermark() -> int()

获取系统内存使用阀值

set_sysmem_high_watermark(Float) -> ok

设置系统内存使用阀值

get_helper_timeout() -> Seconds

获取监测数据返回等待时间

set_helper_timeout(Seconds) -> ok

设置监测数据返回等待时间

从lib/megaco/src/tcp/megaco_tcp_connection.erl摘抄的代码, 挺详细的关于系统的信息:

```erlang
SchedId      = erlang:system_info(scheduler_id),   
SchedNum     = erlang:system_info(schedulers),   
ProcCount    = erlang:system_info(process_count),   
ProcLimit    = erlang:system_info(process_limit),   
ProcMemUsed  = erlang:memory(processes_used),   
ProcMemAlloc = erlang:memory(processes),   
MemTot       = erlang:memory(total),   
error_msg("abormal termination: "  
          "~n   Scheduler id:                         ~p"  
          "~n   Num scheduler:                        ~p"  
          "~n   Process count:                        ~p"  
          "~n   Process limit:                        ~p"  
          "~n   Memory used by erlang processes:      ~p"  
          "~n   Memory allocated by erlang processes: ~p"  
          "~n   The total amount of memory allocated: ~p"  
          "~n~p",   
          [SchedId, SchedNum, ProcCount, ProcLimit,   
           ProcMemUsed, ProcMemAlloc, MemTot, Reason]),   
```


