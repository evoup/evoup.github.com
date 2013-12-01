---
layout: post
title: "批量删除CVS目录的cmd"
date: 2008-07-20 23:23
comments: true
categories:  shell 
---
windows下的bat的方法
```bat
@echo On
@Rem 删除CVS版本控制目录
@PROMPT [Com]#
@echo Find CVS

@for /r . %%a in (.) do @if exist "%%a\CVS" @echo "%%a\CVS"

@echo Find CVS Dir....OK
@pause

@for /r . %%a in (.) do @if exist "%%a\CVS" rd /s /q "%%a\CVS"

@echo Clear CVS Dir Mission Completed

@pause
```


bash的话可以这么搞，参见<a href="http://stackoverflow.com/questions/1330136/script-to-recursively-delete-cvs-directory-on-server">http://stackoverflow.com/questions/1330136/script-to-recursively-delete-cvs-directory-on-server</a>
```bash
#!/bin/sh

if [ -z "$1" ]; then
    echo "Usage: $0 path"
    exit 1
fi

find "$1" -name 'CVS' -type d -print0 | xargs -0 rm -Rf
# or find … -exec like you have, if you can't use -print0/xargs -0
# print0/xargs will be slightly faster.
```