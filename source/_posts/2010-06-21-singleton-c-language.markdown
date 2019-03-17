---
layout: post
title: "单例模式C语言版"
date: 2010-06-21 07:38
comments: true
categories:           c-language
---

Tokyo Cabinet的源码看到的

```c
/* Get the global memory pool object. */
TCMPOOL *tcmpoolglobal(void){
    if(tcglobalmemorypool) return tcglobalmemorypool;//如果有全局内存池对象就返回对象
    tcglobalmemorypool = tcmpoolnew();//如果没有就创建啊^_^
    atexit(tcmpooldelglobal);
    return tcglobalmemorypool;
}
```

而这个tcglobalmemorypool，其实是写在全局的，有

```c
/* Global memory pool object. */
TCMPOOL *tcglobalmemorypool = NULL;
```

后记：其实只要把tcglobalmemorypool定义为全局变量，是个人都会创建这种模式的吧...
