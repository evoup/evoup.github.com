---
layout: post
title: "windows：Gvim关闭声音警报和屏闪"
date: 2012-10-26 12:07
comments: true
categories: vim 
---
```vim
if has("win32")
  set visualbell t_vb=  "关闭visual bell
  au GuiEnter * set t_vb= "关闭beep
endif
```
