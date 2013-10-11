---
layout: post
title: "mysql_fetch_array不使用while"
date: 2009-06-10 13:14
comments: true
categories:          php
---
原来mysql_fetch_array也可以直接就分配到一个array，不用while
方法是
```php
$num=mysql_num_rows($result);
if ($num==0) return 0;
for($i=0;$i<$num;$i++)
{
    $row[]=mysql_fetch_array($result);
}
```