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
$allFiles=array('/path/to/file0','/path/to/file1');
foreach($allFiles as $file) {
    $temstr=file_get_contents($file);
    if (!function_exists('mb_detect_encoding')) {
        echo "请安装mb函数集\n";
        exit;
    }
    $encode = mb_detect_encoding($temstr,"ASCII,UTF-8,CP936,EUC-CN,BIG-5,EUC-TW");
    if ($encode!="UTF-8") {
        echo "{$file}文件编码从{$encode}转换为UTF-8\n";
        $temstr=mb_convert_encoding($temstr, "UTF-8", $encode);
        file_put_contents($file,$temstr);
    }
}
```

