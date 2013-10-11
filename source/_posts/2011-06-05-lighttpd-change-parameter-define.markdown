---
layout: post
title: "C语言变参宏"
date: 2011-06-05 23:29
comments: true
categories:     c-language
---
研究lighttpd1.4.28代码的时候，到缓存调用部分，有这么一句：

```c
buffer_copy_string_len(modules->key, CONST_STR_LEN("server.modules"));
```
而此参数声明的时候是这样的

```c
int buffer_copy_string_len(buffer *b, const char *s, size_t s_len);
```
怎么是三个参？从CONST_STR_LEN入手，这是一个宏

```c
#define CONST_STR_LEN(x) x, x ? sizeof(x) - 1 : 0
```
这不就成了三个参了？记一笔...