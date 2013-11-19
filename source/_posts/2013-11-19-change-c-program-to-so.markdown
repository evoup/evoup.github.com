---
layout: post
title: "如何正确地把带参数的C语言main程序改成so"
date: 2013-11-19 18:17
comments: true
categories: c-language 
---

首先有些标题党吧，不过个人体会修改程序的过程中遇到未知的坑还是比较阴险的，所以整理一下写个博客。

之前尝试把C语言的带参数执行的main程序改成动态链接库，发生了一个问题，主程序调用动态链接库最后获取的结果保持不变，跑了一会儿之后还是和第一次调用的一样。把几乎可能导致问题的static变量全部给改为非静态变量和重置后居然还是无效。最后在痛苦的查询资料之后，终于找到了问题所在。

话说回来，先看怎么把带参main程序改成动态链接库。

<!-- more -->

先看准备被改成动态库的原代码
```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
int main(int argc, char **argv) {
    int ch;
    const char *opts="a:b::cd";
    while ((ch = getopt(argc, argv, opts)) != -1) {
        switch (ch) {
        case 'a':
            printf("option a:'%s'\n",optarg);
            break;
        case 'b':
            printf("option b:'%s'\n",optarg);
            break;
        case 'c':
            printf("option c:'%s'\n",optarg);
            break;
        case 'd':
            printf("option d:'%s'\n",optarg);
            break;
        default:
            printf("other option:%c\n",ch);
        }
    }
}
```

```c
gcc -Wall test.c -o test
```
测试：
```sh
./test -a1 -b1 -c -d
option a:'1'
option b:'1'
option c:'(null)'
option d:'(null)'
```
稍微讲下getopt的用法，这个命令是提供命令行执行可执行程序带不同参数的功能实现。以上程序根据所数入的参数，执行相应的操作。
其中opts的a:b::cd，“:”表示必须该选项带有额外的参数，全局变量optarg会指向此额外参数，“::”标识该额外的参数可选(有些Uinx可能不支持“::”）。

接下来把它给改成SO，然后再主调程序中直接指定参数进行调用。
```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
int dlmain(int, char **);
int main(int argc, char **argv) {
    char *cmdStr[5]={"","-a1","-b1","-c","-d\0"};
    dlmain(5,cmdStr);
    return 1;
}
int dlmain(int argc, char **argv) {
    int ch;
    const char *opts="a:b::cd";
    while ((ch = getopt(argc, argv, opts)) != -1) {
        switch (ch) {
        case 'a':
            printf("option a:'%s'\n",optarg);
            break;
        case 'b':
            printf("option b:'%s'\n",optarg);
            break;
        case 'c':
            printf("option c:'%s'\n",optarg);
            break;
        case 'd':
            printf("option d:'%s'\n",optarg);
            break;
        default:
            printf("other option:%c\n",ch);
        }
    }
    printf("in dll\n");
    return 1;
}
```
测试一下
```sh
gcc -Wall plug.c -o plug
./plug
option a:'1'
option b:'1'
option c:'(null)'
option d:'(null)'
in dll
```
ok，没有问题，直接转换成动态库
```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
int d_plug ();
int dlmain(int, char **);
int d_plug () {
    char *cmdStr[5]={"","-a1","-b1","-c","-d\0"};
    dlmain(5,cmdStr);
    return 1;
}
int dlmain(int argc, char **argv) {
    int ch;
    const char *opts="a:b::cd";
    while ((ch = getopt(argc, argv, opts)) != -1) {
        switch (ch) {
        case 'a':
            printf("option a:'%s'\n",optarg);
            break;
        case 'b':
            printf("option b:'%s'\n",optarg);
            break;
        case 'c':
            printf("option c:'%s'\n",optarg);
            break;
        case 'd':
            printf("option d:'%s'\n",optarg);
            break;
        default:
            printf("other option:%c\n",ch);
        }
    }
    printf("in dll\n");
    return 1;
}
```

编译动态连接库
```sh
gcc -shared -o plug.so plug.c -fpic
```
这样就得到了plug.so

接下来写主调程序，用它直接调用已经通过改两行代码就封装好的动态库。
```c
#include <stdio.h>
#include <dlfcn.h>
int (*d_plug) ();
int main(int argc,char **argv) {
    void *dp;
    dp=dlopen("./plug.so",RTLD_LAZY);
    d_plug=dlsym(dp,"d_plug");
    d_plug();
    dlclose(dp);
    return 1;
}
```
编译执行看结果
```sh
gcc main.c -lc -fpic -o main
./main
option a:'1'
option b:'1'
option c:'(null)'
option d:'(null)'
in dll
```

###其他注意事项：
其实除了optarg，还存在全局变量optind、opterror和optopt。三者的作用

optarg：非常明了，就是指程序的参数，是个字符串指针

optind：是下一次调用getopt的时，从optind存储的位置处重新开始检查选项

opterr：当opterr=0时，getopt不向stderr输出错误信息

optopt: 当命令行选项字符不包括在optstring中或者选项缺少必要的参数时，该选项存储在optopt 中，getopt返回'？’


###问题解决
写erlang的NIF扩展时optind需要重置为0，否则就导致了每次结果不变，因为就我所知erlang目前版本的NIF是没有unload功能的。

