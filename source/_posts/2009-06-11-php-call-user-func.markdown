---
layout: post
title: "php_call_user_func"
date: 2009-06-11 11:40
comments: true
categories:     php
---
call_user_func这个函数，可以把函数存到数组之后，在代码的任何位置进行调用，非常方便。

在a2billing/Public/call-log-custoners.php的大约780行


```php
while...
echo $FG_TABLE_COL[$i][11];
call_user_func($FG_TABLE_COL[$i][11], $record_display);
}else{
echo stripslashes($record_display);
}
...
end while
```

这里$FG_TABLE_COL[$i][11]其实是放的是函数，设计目的是循环echo一行的每个列，如果
$FG_TABLE_COL[$i][11]里有函数，就用函数去格式化$record_display!!!!
call_user_func这是PHP的函数，我还以为是自定义函数！
到把自定义函数都放在数组里，然后用call_user_func，第一个参数设置为
数组，第二个参数设置为自定义函数的参数。这个设计思想很直接借鉴。

后计：这种设计方法，比较适合用在设计restful程序的框架上。