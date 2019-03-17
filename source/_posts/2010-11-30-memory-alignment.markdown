---
layout: post
title: "目前看到的对内存对齐问题最好的解释"
date: 2010-11-30 12:14
comments: true
categories:     c-language
---
简单的说，对于一个class或是struct：

按里面所占字节最大的类型为位域任何变量的存储不能跨位域 比如：

<!-- more -->

对于

```c
struct A {
int a;
char c;
int b;
}
```

以int类型所占字节为位域，也就是4。然后，对于a，四个字节。对于c，一个字节，后面三个字节不能用来存储b，因为b是四个，如果用这三个，那么必将剩余一个，根据原则，不可跨位域。

所以存放形式：

```bash
aaaaaaaa aaaaaaaa aaaaaaaa aaaaaaaa

cccccccc XXXXXXXX XXXXXXXX XXXXXXXX

bbbbbbbb bbbbbbbb bbbbbbbb bbbbbbbb
```
