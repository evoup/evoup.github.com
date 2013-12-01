---
layout: post
title: "nginx反向代理mongodb的web页面"
date: 2013-01-05 15:32
comments: true
categories:  [nginx,mongodb] 
---

需要配置nginx来反向代理出mongodb的web页面，参考这篇文章
<!-- more -->
http://serverfault.com/questions/418212/nginx-reverse-proxy-to-mongodb-rest-interface



{% codeblock nginx配置文件 lang:bash %}
...
location / {

       access_log /var/log/access.log access1;

       proxy_redirect     off;

       proxy_connect_timeout      90;

       proxy_send_timeout         90;

       proxy_read_timeout         90;

       proxy_buffer_size          4k;

       proxy_buffers              432k;

       proxy_busy_buffers_size    64k;

       proxy_temp_file_write_size 64k;

       add_header Cache-Control no-cache;

       proxy_pass http://172.16.30.184:28017;

   }

}
...
{% endcodeblock %}
不足：只能从根目录进行代理,如需更加完善，需要装编译nginx的时候， --with-http_stub_status_module

然后参考《nginx反代加替换傻瓜教程》进行配置

