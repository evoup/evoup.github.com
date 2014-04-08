---
layout: post
title: "freebsd下nginx+Php5.3.8+php-fpm源码安装之二"
date: 2013-04-08 16:33
comments: true
categories: [php,nginx]
---


nginx端的配置
首先创建好web目录，假设在/services/www目录，在其下创建一个php测试脚本，php的脚本简单如下：

```
<?php
phpinfo();
```

安装nginx
安装简单的采用port，然后直接配置/usr/local/etc/nginx/nginx.conf

添加一个前缀test，我们访问http://192.168.216.198/test/index.php
或者
http://192.168.216.198/test/
即可看到php输出的信息。

默认nginx的fastcgi脚本似乎不直接支持php，需要改成类似如下的方式：
```sh
#php_fcgi_params.conf                                                                                                                                              
fastcgi_param  GATEWAY_INTERFACE  CGI/1.1;                                                                                                                   
fastcgi_param  SERVER_SOFTWARE    nginx;                                                                                                                     
fastcgi_param  QUERY_STRING       $query_string;                                                                                                             
fastcgi_param  REQUEST_METHOD     $request_method;                                                                                                           
fastcgi_param  CONTENT_TYPE       $content_type;                                                                                                             
fastcgi_param  CONTENT_LENGTH     $content_length;                                                                                                           
fastcgi_param  SCRIPT_FILENAME    $document_root$fastcgi_script_name;                                                                                        
fastcgi_param  SCRIPT_NAME        $fastcgi_script_name;                                                                                                      
#fastcgi_param  SCRIPT_NAME        $request_uri;                                                                                                             
fastcgi_param  PATH_INFO          $fastcgi_script_name;                                                                                                      
fastcgi_param  REQUEST_URI        $request_uri;                                                                                                              
fastcgi_param  DOCUMENT_URI       $document_uri;                                                                                                             
fastcgi_param  DOCUMENT_ROOT      $document_root;                                                                                                            
fastcgi_param  SERVER_PROTOCOL    $server_protocol;                                                                                                          
fastcgi_param  REMOTE_ADDR        $remote_addr;                                                                                                              
fastcgi_param  REMOTE_PORT        $remote_port;                                                                                                              
fastcgi_param  SERVER_ADDR        $server_addr;                                                                                                              
fastcgi_param  SERVER_PORT        $server_port;                                                                                                              
fastcgi_param  SERVER_NAME        $server_name;                                                                                                              
# PHP only, required if PHP was built with --enable-force-cgi-redirect                                                                                       
fastcgi_param  REDIRECT_STATUS    200;

fastcgi_param       HTTP_X_REQUESTED_WITH       $http_x_requested_with;

```

在nginx.conf中的server上下文中添加
```
        location ~ ^/pma/.*$ {
            #access_log /var/log/access_beta.log access1;
            index index.php index.html index.htm;
            root /services/www/;
            fastcgi_index   index.php;
            include /usr/local/etc/nginx/php_fcgi_params.conf;
            fastcgi_pass 127.0.0.1:9000;
        }
```

启动nignx
```sh
$ /usr/local/etc/rc.d/ngnix start
```

访问浏览器http://192.168.216.198/pma/

![Alt text](/images/evoup/php-fpm-pma.png)

done!
