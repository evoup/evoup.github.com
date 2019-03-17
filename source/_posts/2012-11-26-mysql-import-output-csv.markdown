---
layout: post
title: "浅谈mysql导入导出csv的几种方法"
date: 2012-11-26 09:56
comments: true
categories: mysql 
---

有几种方法，这里一一列举：

###第一种：使用phpmyadmin选择文件上传
此种方式很直观，只是要注意选择正确的换行符。假设我们在mysql的shell中采用如下方式先导出数据

```sql
select * from test_info
into outfile '/tmp/test_info.csv'
fields terminated by ',' optionally enclosed by '"' escaped by '"'
lines terminated by '\r\n';
```

则需要上传test_info.csv，点击CSV using LOAD DATA，然后按照如下设置进行导入
（其中fields terminated为分隔字段，optionally enclosed为可选包裹字段，escaped为转移字段，line terminated为行终止字段）

![Alt text](/images/evoup/phpmyadmin.png)

###第二种：mysql shell
其实就是刚刚导出的逆操作。在mysql的shell中操作：

```mysql
load data infile '/tmp/test_info.csv'
	into table test_info 
	fields terminated by ','  optionally enclosed by '"' escaped by '"'
	lines terminated by '\r\n';
```

###第三种：php程序解析后插入

```php
<?php
echo "开始导入csv数据到工作表\n";
echo "导入test_info...\n";
$mysql_host="172.16.27.55";
$mysql_user="user";
$mysql_pass="pass";
$db="test_info";
$handle = fopen ('/tmp/test_info.csv','r');  
$link= mysqli_init();
$link->options(MYSQLI_OPT_CONNECT_TIMEOUT, 8);
$link->real_connect($mysql_host, $mysql_user, $mysql_pass, $db);
$link->query("SET NAMES utf8");
$query="insert into `test_info` (`字段1`,`字段2`) values (";  
while ($data = fgetcsv ($handle)) {  
    $num = count ($data);  
    for ($i=0; $i<$num; $i++) {  
        if($i == $num-1){  
            $query .= "\"".$data[$i]."\")";  
            break;  
        }  
        $query .= "\"".$data[$i]."\",";  
    }  

    if (!($result=$link->query($query))) {
        echo $query;
        echo ("[导入失败]\n");
        exit;
    }

$query="insert into `test_info` (`字段1`,`字段2`) values (";  

}
?>
```

需要注意如果fgetcsv出错和到了行尾都是会返回false的。
