---
layout: post
title: "freebsd下PHP的Tokyo Tyrant安装配置测试"
date: 2013-09-26 16:24
comments: true
categories:  [php,nosql]
---
###简介和横向对比
Tokyo Tyrant（ttserver）是一款分布式nosql数据库，内部主要是用Tokyo Cabinet数据库实现。与memcache相比，其具有持久化存储的特性；与redis相比，这个数据库在记录数上亿时性能会急剧下降，没有redis业界口碑好，当然也可能是因为这是比较早起的nosql产品了，以及没有进行正确的配置导致的。除此之外ttserver还支持memcache兼容协议，仅仅需要注意INCR的操作中，memcache是不返回值，ttserver是返回自增ID的。

<!-- more -->

###php扩展的安装
编译的方式还是分静态方式和动态方式，特点是静态编译之后运行效率较高，但是需要重新编译php源码，动态编译只需要编译出动态链接库，然后在php.ini上加载so=XXXX.so即可。需要知晓php的版本在5.2以上（实际测试发现php的版本也不能太高，php5.2.10测试通过）,此外编译前必须先安装tt，否则会收到` configure: error: Please reinstall the Tokyo Tyrant distribution
的报错。
####安装编译依赖
```sh
$ cd /usr/ports/databases/tokyotyrant
$ sudo make install clean
```
####静态方式
```sh
$ cd /home/software
$ fetch http://museum.php.net/php5/php-5.2.10.tar.bz2
$ tar xjf php-5.2.10.tar.bz2
$ cd php-5.2.10/ext
$ fetch http://pecl.php.net/get/tokyo_tyrant-0.7.0.tgz
$ tar xzf tokyo_tyrant-0.7.0.tgz
$ mv tokyo_tyrant-0.7.0 tokyo_tyrant
$ cd ..
$ ./buildconf --force
$ ./configure --prefix=/usr/local/php_tt --with-tokyo-tyrant
$ make
$ sudo make install
``` 
查看模块是否成功安装
```sh
$ /usr/local/php_tt/bin/php -m | grep tokyo
tokyo_tyrant
```

####动态方式(假设php已经安装在/usr/local/php_tt)
```sh
$ cd /home/software
$ fetch http://pecl.php.net/get/tokyo_tyrant-0.7.0.tgz
$ tar xzf tokyo_tyrant-0.7.0.tgz
$ cd tokyo_tyrant-0.7.0
$ /usr/local/php_tt/bin/phpize
$ ./configure --with-php-config=/usr/local/php_tt/bin/php-config
$ make
$ sudo make install
```
然后复制刚才的编译好的so文件到扩展目录到etc目录
```sh
$ cp /usr/local/php_tt/lib/php/extensions/no-debug-non-zts-20090626/tokyo_tyrant.so /usr/local/php_tt/etc/
```

编辑php的配置文件
```sh
vi /usr/local/php_tt/etc/php.ini
```
加入1行
```sh
extension=tokyo_tyrant.so
```


####测试1
编辑php文件
```php
<?php
/**
 * test.php
 */
$tt = new TokyoTyrant("localhost");
$tt->put("language", "C/C++");
echo 'language:' . $tt->get("language") . "\n";

echo 'The count of records is:' . $tt -> num() . "\n";

print_r($tt -> stat());

$it = $tt -> getIterator();
foreach ($it as $key => $val) {
    echo "key: $key, val: $val\n";
}
?>
```
运行
```sh
/usr/local/php_tt/bin/php test.php
language:C/C++
The count of records is:1
Array                                                                                                                                                [0/1804]
(
    [version] => 1.1.41
    [libver] => 324
    [protver] => 0.91
    [os] => FreeBSD
    [time] => 1388045682.650761
    [pid] => 69442
    [sid] => 0
    [type] => on-memory
    [path] => *
    [rnum] => 1
    [size] => 525437
    [bigend] => 0
    [fd] => 5
    [loadavg] => 0.958008
    [memrss] => 92448
    [ru_user] => 0.109603
    [ru_sys] => 0.137003
    [ru_real] => 352.795682
    [cnt_put] => 2
    [cnt_putkeep] => 0
    [cnt_putcat] => 0
    [cnt_putshl] => 0
    [cnt_putnr] => 0
    [cnt_out] => 0
    [cnt_get] => 3
    [cnt_mget] => 0
    [cnt_vsiz] => 0
    [cnt_iterinit] => 2
    [cnt_iternext] => 2
    [cnt_fwmkeys] => 0
    [cnt_addint] => 0
    [cnt_adddouble] => 0
    [cnt_ext] => 0
    [cnt_sync] => 0
    [cnt_optimize] => 0
    [cnt_vanish] => 0
    [cnt_copy] => 0
    [cnt_restore] => 0
    [cnt_setmst] => 0
    [cnt_rnum] => 2
    [cnt_size] => 0
    [cnt_stat] => 2
    [cnt_misc] => 0
    [cnt_repl] => 0
    [cnt_put_miss] => 0
    [cnt_out_miss] => 0
    [cnt_get_miss] => 0
)
key: language, val: C/C++
```
可以看到这个扩展的遍历功能相当好用:)

####参考
linux下的请自行参考《Debian下PHP的Tokyo Tyrant安装配置及测试》




