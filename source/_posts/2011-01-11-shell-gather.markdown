---
layout: post
title: "shell收集"
date: 2011-1-11 12:11
comments: true
categories: shell 
---

批量拷贝文件到服务器bash版
```bash
#!/usr/local/bin/bash
for hosts in `cat host.list` ;
do
    echo "将处理"$hosts ;
    scp /services/monitor_deal/madmonitor.current root@$hosts:/services/monitor_deal/ ;
    scp /services/monitor_deal/libphp5.so root@$hosts:/usr/local/lib/ ;
    scp /services/monitor_deal/libmysqlclient.so.16 root@$hosts:/usr/local/lib/ ;
    scp /services/monitor_deal/libz.so.6 root@$hosts:/usr/lib/ ;
done

```

远程备份MySQL数据库
```sh
#备份整个库
mysqldump -uusername -p*** -h 192.168.1.123 dbname > db_backup.sql

#备份单个表
mysqldump -uusername -p*** -h 192.168.1.123 dbname tablename > table_backup.sql

#导入整个库
mysql -uusername -p*** dbname < db_backup.sql

#导入单个表(2014-4-6补充：windows下可以这样c:\> mysql -uusername -p*** dbname < tablename < f:/table_backup.sql)
mysql -uusername -p*** dbname < table_backup.sql
```
备注：其实也可以用source，不过要进入mysql的shell中操作。

取得fb6的网卡mac
```sh
ifconfig lo0 | grep '[0-9a-f]\{8\}' | awk '{print $4}'
```

查看目录里的每个子目录的大小
```sh
#for freebsd
du -h -d 1
#for linux
du -h –max-depth=1 
```

获取文本的第三行
```sh
sed -n '3p' file 
```

截取字符串
```sh
echo "abcd" | cut -b 1-3
```

使用nc访问百度
```sh
printf "GET / HTTP/1.0\r\n\r\n" | nc www.baidu.com 80
```

移除UTF-8的BOM
```sh
 sed -e '1s/^\xef\xbb\xbf//' < bomfile > newfile
```

无输出在后台静默执行命令
```sh
nohup sh command >> /dev/null
#注意如果command是一个脚本，所在脚本可以写成&后台执行的方式以获得主调代码异步的机制
```

vim不加载vimrc
```sh
vim -u NONE
```

sudo的时候使用本用户的vimrc配置
```sh
sudo 
```

调用vim来把文件转换为utf-8格式，注意要和find连用，需加载vimrc不然乱码
```sh
vim -c "set fileencoding=utf-8 | wq!" main.cpp
```
find查找c/c++的源文件和头文件
```sh
find . -type f \( -name "*.cpp" -o -name "*.h" -o -name "*.c" \)
```

获取当前时间戳
```sh
date "+%s"
```
不用scp，而使用rsync把本地文件传到远程服务器的一个目录
```sh
rsync -avze 'ssh -p 2203' logfile user@172.16.30.112:/logfiles/
```

把时间戳转换成系统时间
```sh
#linux版本的
date -d '1970-01-01 UTC 946684800 seconds' +"%Y-%m-%d %T %z"
#freebsd版本的
date -r 1384499085 +"%Y-%m-%d %T %z"
```
按压缩包内的文件名,从bz2格式精确解压出指定文件
```sh
//解压bz2CompressedFile文件，提取其中的文件fileName重定向到destFile
bzcat bz2CompressedFile | tar xOf - fileName > destFile
```

在第三行之后插入指定数据
```awk
awk '{v1="###\n\呵呵呵";if (NR==4)print v1;print}' file > file_translated
```
参考这里http://club.topsage.com/thread-357787-1-1.html


下面这个等同于find . | xargs grep error 但是可以同时打印error和fail为关键字匹配的行
```awk
awk '/(error|fail)/ { print; }' syslog.log
```

统计源码总行数
```sh
find . -exec wc -l {} \; | awk '{print $1}'
find . -type f ! -path "*.svn*" -exec wc -l {} \; | awk '{sum=sum+$1}END{print sum }'
```

如果要给sum来个初值，那么
```sh
find . ! -path "*.svn*" -exec wc -l {} \; | awk '{sum=0}{sum=sum+$1}END{print sum }'
```

寻找全部process开头的rrd文件，删除前确认
```sh
find . -name "process*.rrd" -ok rm {} \;
```

打印某网卡信息
```sh
ifconfig | awk '{if(NR==5) print $2}'
```

整个文件夹内替换字符串
```sh
find . -name "*.php" -exec sed -i '' -e 's/checkDigital/validDigital/g' {} +
```

找到的c头文件放到temp目录下
```sh
find . -name "*.h" -exec mv {} temp/ \;
```

AWK的入门

http://www.chemie.fu-berlin.de/chemnet/use/info/gawk/gawk_3.html

http://blog.csdn.net/eroswang/archive/2009/04/11/4064325.aspx

AWK高级用法

http://blog.csdn.net/eroswang/archive/2010/01/26/5258216.aspx

AWK一句话手册

http://blog.csdn.net/yangyinbo/archive/2010/05/12/5583936.aspx

可以用得到的shell的脚本

http://blog.csdn.net/eroswang/archive/2010/04/16/5494482.aspx

cshell参考

http://tech.it168.com/KnowledgeBase/Articles/4/b/9/4b910acff687a1096011b7ce80d3b59e.htm

数组的定义
```bash
set array=(a b c d)
居然下标是从1开始
echo ${A[1]}  
a
```
bash下实现的伪多进程(实际是用了&后台执行)
```bash
　　#!/bin/bash

　　for ((i=0;i<5;i++));do

　　{

    　　sleep 3;echo 1>>aa && echo "done!"

　　} &

　　done

　　wait

　　cat aa|wc -l

　　rm aa
```

bash的fifo实现的多线程

http://www.examda.com/linux/fudao/20101011/132107473.html

多线程的实现是采用了有名管道mkfifo

http://blog.csdn.net/yangyun1981/archive/2007/12/19/1954061.aspx

shell判断文件是否存在

http://hi.baidu.com/hy0kl/blog/item/03737a34aad2795e241f1431.html
