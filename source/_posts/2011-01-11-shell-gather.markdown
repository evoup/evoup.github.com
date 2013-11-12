---
layout: post
title: "shell收集"
date: 2011-1-11 12:11
comments: true
categories: shell 
---

取得fb6的网卡mac
```sh
ifconfig lo0 | grep '[0-9a-f]\{8\}' | awk '{print $4}'
```

在第三行之后插入指定数据
```awk
awk '{v1="###\n\呵呵呵";if (NR==4)print v1;print}' file > file_translated
```
参考这里http://club.topsage.com/thread-357787-1-1.html


find最后改成了这样
```sh
find . \( -path '*.svn*' -o -path '*.fcgi*' -o -path '*tags*' \) -prune -o -print | xargs grep -i
```

sh里面没有set,而linux下则是链接到了bash 
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

打印某网卡信息
```sh
ifconfig | awk '{if(NR==5) print $2}'
```

server1上执行传输，即可完成从server1克隆sda硬盘到server2的任务
```sh
# nc -l -p 1234 | dd of=/dev/sda
```

整个文件夹内替换字符串
```sh
find . -name "*.php" -exec sed -i '' -e 's/checkDigital/validDigital/g' {} +
```
