---
layout: post
title: "js获取cookie"
date: 2008-05-22 16:31
comments: true
categories:    javascript
---
直接参考网易的函数
```javascript
function getCookie(name) {
   var search = name + "="
   if(document.cookie.length > 0) {
      offset = document.cookie.indexOf(search)
      if(offset != -1) {
         offset += search.length
         end = document.cookie.indexOf(";", offset)
         if(end == -1) end = document.cookie.length
         return unescape(document.cookie.substring(offset, end))
      }
      else return ""
   }
```

<hr>
后计，直接使用jquery的cookie插件更方便，取一个cookie(name)赋值给foo
```javascript
var foo= $.cookie('name');
```