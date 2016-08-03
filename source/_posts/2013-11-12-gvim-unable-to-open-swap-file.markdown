---
layout: post
title: "gvim无法打开交换文件"
date: 2013-11-12 12:21
comments: true
categories: vim 
---

最近在windows下使用gvim，直接按照之前在freebsd上的配置搬回来，发现问题多多。每次打开文件都会报错E303，非常碍事。

```vim
:help E303
Unable to open swap file for "{filename}", recovery impossible

Vim was not able to create a swap file.  You can still edit the file, but if
Vim unexpected exits the changes will be lost.  And Vim may consume a lot of
memory when editing a big file.  You may want to change the 'directory' option
to avoid this error.  See |swap-file|.
```
说是无法创建交换文件。
其实只要这么解决：

```vim
:set directory=.,$TEMP
```

或者指定一个实际存在路径。
