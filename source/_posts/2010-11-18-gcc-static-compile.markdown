---
layout: post
title: "gcc静态编译"
date: 2010-11-18 16:50
comments: true
categories:           gcc
---
制作静态动态库参考这篇文章《Creating a shared and static library with the gnu compiler [gcc]》
http://www.adp-gmbh.ch/cpp/gcc/create_lib.html
编译静态库
先找一下源文件
```bash
find . -name "*.c"
./sfhashfcn.c
./test.c
./sfmemcap.c
./sfxhash.c
./sfprimetable.c
```
生成中间文件
```bash
gcc -c sfhashfcn.c -o sfhashfcn.o
gcc -c sfmemcap.c -o sfmemcap.o
gcc -c sfxhash.c -o sfxhash.o
gcc -c sfprimetable.c -o sfprimetable.o
```
生成静态库
```bash
ar rcs libthash.a sfmemcap.o sfxhash.o sfprimetable.o sfhashfcn.o
```
最后链接（注意libxx.a，最后链接参数是-lxx）
```bash
gcc -static test.c -L. -lthash -o statically_linked_test
```

如果c++的项目要使用c的静态库则需要在引用的头文件的外面加上如下的代码
```c
extern "C" {
    #include "header.h"
}
```