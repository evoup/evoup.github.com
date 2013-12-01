---
layout: post
title: "完美解决phpmyadmin4.1.0访问缓慢"
date: 2013-11-29 10:52
comments: true
categories: [php,mysql]
---
安装最新版phpmyadmin4.1.0出现了访问非常缓慢的问题，记录一下

<!-- more -->

###第一步
文件名./libraries/Util.class.php文件
查找
```php
return strftime($date, $timestamp);
```
替换为
```php
if(extension_loaded('gettext'))
  return strftime($date, $timestamp);
```

###第二步
文件名./version_check.php文件
注释掉以下代码
```php
$version = PMA_Util::getLatestVersion();

echo json_encode(
    array(
        'version' => $version->version,
        'date' => $version->date,
        )
    );
```
###第三步
直接浏览器访问/setup路径，进行设置

###第四步
应该可以登录了，如果不行清除cookie应该也可以访问了。
