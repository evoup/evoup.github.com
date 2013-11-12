---
layout: post
title: "解决gvim菜单乱码问题"
date: 2013-12-12 11:15
comments: true
categories: vim 
---
windows下的gvim在使用过程中还有一个问题，就是当我设置了set encoding=utf-8之后，菜单就出现了乱码。如果不设置呢，那么编辑文件就不能是utf-8，网上搜了一圈之后，终于发现比较满意的解决方法。

<!-- more -->

```
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
"vim7.1在windows下的编码设置。
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

set encoding=utf-8

set fileencodings=utf-8,chinese,latin-1

if has("win32")

 set fileencoding=chinese

else

 set fileencoding=utf-8

endif

"解决菜单乱码

source $VIMRUNTIME/delmenu.vim

source $VIMRUNTIME/menu.vim

"解决consle输出乱码

language messages zh_CN.utf-8
```
