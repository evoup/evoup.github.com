---
layout: post
title: "java关于ConcurrentModificationException异常"
date: 2009-12-23 15:46
comments: true
categories: java
---

如果出现java.util.ConcurrentModificationException

主要原因是使用了叠代器，而且删除了某元素。临时的解决方法是设置1个标记，如果遇到该标记略过！