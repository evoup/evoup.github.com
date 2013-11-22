---
layout: post
title: "php转换文件编码"
date: 2010-11-22 17:56
comments: true
categories: php 
---
工作中需要在代码里把文件编码先判断其格式，然后转换为UTF-8直接上代码了
<!-- more -->
```php
$encode = mb_detect_encoding($temstr,"ASCII,UTF-8,CP936,EUC-CN,BIG-5,EUC-TW");
if ($encode!="UTF-8") {
    echo "{$file}文件编码从{$encode}转换为UTF-8\n";
    $temstr=mb_convert_encoding($temstr, "UTF-8", $encode);
    file_put_contents($file,$temstr);
}
```

