---
layout: post
title: "erlang R15B01安装问题"
date: 2013-06-19 14:21
comments: true
categories: erlang 
---
freebsd8.0上安装erlang15B01，报错,可以直接修改vi ./erts/emulator/i386-unknown-freebsd8.1/Makefile的-DUSE_VM_PROBES参数，指定其=0绕过。
