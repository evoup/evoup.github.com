---
layout: post
title: "php myadmin1045 error"
date: 2010-11-19 15:58
comments: true
categories: php 
---

今天遇到phpMyAdmin错误 #1045 - Access denied for user@localhost的错误

记得在./libraries/config.default.php
已经制定好了需要的变量。 
```php
 57 $cfg['Servers'][$i]['host']          = '172.16.27.55'; // MySQL hostname or IP address
 58 $cfg['Servers'][$i]['port']          = '3306';          // MySQL port - leave blank for default port
 59 $cfg['Servers'][$i]['socket']        = '';          // Path to the socket - leave blank for default socket
 60 $cfg['Servers'][$i]['connect_type']  = 'tcp';       // How to connect to MySQL server ('tcp' or 'socket')
 61 $cfg['Servers'][$i]['extension']     = 'mysql';     // The php MySQL extension to use ('mysql' or 'mysqli')
 62 $cfg['Servers'][$i]['compress']      = FALSE;       // Use compressed protocol for the MySQL connection
 63                                                     // (requires PHP >= 4.3.0)
 64 $cfg['Servers'][$i]['controluser']   = '';          // MySQL control user settings
 65                                                     // (this user must have read-only
 66 $cfg['Servers'][$i]['controlpass']   = '';          // access to the "mysql/user"
 72 $cfg['Servers'][$i]['user']          = 'user';      // MySQL user
 73 $cfg['Servers'][$i]['password']      = '%**';          // MySQL password (only needed
```



同时
```php
 40  * The 'cookie' auth_type uses blowfish algorithm to encrypt the password. If
 41  * at least one server configuration uses 'cookie' auth_type, enter here a
 42  * passphrase that will be used by blowfish. The maximum length seems to be 46
 43  * characters.
 44  */
 45 $cfg['blowfish_secret'] = 's#$dcrnm^9fk2';
```
这部分也指定完毕。

但是还是会报告1045，最后清除浏览器缓存，终于搞定。


