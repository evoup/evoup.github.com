---
layout: post
title: "nginx with rrdcgi"
date: 2013-10-25 17:27
comments: true
categories: [rrdtool,nginx] 
---

开始在web界面上加载监控图表了，用rrdrool graph生成图，但是发现只能够生成。于是想当然地试了一下rrdcgi，本以为能够出图，结果还是创建图片，html来加载图片。最后发现ganglia的图表中居然也是先提取在临时目录生成好的图片，然后用php来生成头，最后再删除图片。不过顺便把nginx下配置CGI程序的知识学会了，权且记一笔。

###RRDCGI的使用
首先是编写graph.cgi
```bash
#!/usr/local/bin/rrdcgi
 <RRD::GRAPH
      /services/cgi-bin/load.png
      --imginfo '<IMG SRC=/rrdgraph/%s WIDTH=%lu HEIGHT=%lu >'
      --lazy --title="load"
      --start 1382666836 --end 1382677047
      --width 705 --height 245
      --alt-autoscale
          DEF:load=/services/rrds/yin-arch_ac101eb8/load.rrd:load:AVERAGE
          HRULE:1#ff0000:"warning value"
      AREA:load#3d3d3d:load>
```
语法基本和rrdgraph的差不多，没什么好说的，这样等等会生成出来一个html代码叫做
```html
<IMG SRC=/rrdgraph/load.png WIDTH=786 HEIGHT=324 >
```

要怎么在shell中直接验证能出图呢？
```bash
sudo rrdcgi graph.cgi < /dev/null
(offline mode: enter name=value pairs on standard input)
Content-Type: text/html
Content-Length: 53

 <IMG SRC=/rrdgraph/load.png WIDTH=786 HEIGHT=324 >
```
很明显这样子是得到了load.png这个图片文件。然后把这个cgi文件移到/services/cgi-bin/目录下待机。

###nginx的对rrdcgi支持的配置
首先需要安装好perl，还需要用到以下库：（以下版本可能过旧，直接到CPAN的网站搜索安装）
```bash
wget http://www.cpan.org/modules/by-module/FCGI/FCGI-0.67.tar.gz
tar -zxf FCGI-0.67.tar.gz
cd FCGI-0.67
perl Makefile.PL
make && make install
cd ..

wget http://search.cpan.org/CPAN/authors/id/G/GB/GBJK/FCGI-ProcManager-0.18.tar.gz
tar -zxf FCGI-ProcManager-0.18.tar.gz
cd FCGI-ProcManager-0.18
perl Makefile.PL
make && make install
cd ..

wget http://search.cpan.org/CPAN/authors/id/I/IN/INGY/IO-All-0.39.tar.gz
tar zxf IO-All-0.39.tar.gz
cd IO-All-0.39
perl Makefile.PL
make && make install
```
安装 nginx-fcgi 脚本：
```bash
wget http://hily.me/blog/wp-content/uploads/2010/01/nginx-fcgi.txt

mv nginx-fcgi.txt /usr/sbin/nginx-fcgi

chmod +x /usr/sbin/nginx-fcgi
```
如果不用 sudo 方式运行 nginx-fcgi，请注释掉 nginx-fcgi 脚本中的：
```bash
if ( $> == “0″ ) {
print “\n\tERROR\tRunning as a root!\n”;
print “\tSuggested not to do so !!!\n\n”;
exit 1;
}
```
启动 nginx-fcgi：
```bash
sudo nginx-fcgi -l /var/log/nginx/nginx-fcgi.log -pid /var/run/nginx-fcgi.pid -S /var/run/nginx-fcgi.sock
```

注意一定要为 socket 添加 nginx 帐户的权限，否则 cgi 会执行失败。
新建 nginx-fcgi 脚本指令配置，直接从 fastcgi_params 复制模板：
```bash
cp /etc/nginx/fastcgi_params /etc/nginx/nginx_fcgi_params
```

去除尾部的：
```bash
# PHP only, required if PHP was built with –enable-force-cgi-redirect
fastcgi_param  REDIRECT_STATUS    200;
```

最后的cgi站点配置：
```bash
        location ~ ^/cgi-bin/.*\.cgi$
        {
            fastcgi_index  index.cgi;
            fastcgi_param  SCRIPT_FILENAME    /services$fastcgi_script_name;
            include        nginx_fcgi_params;
            fastcgi_read_timeout    5m;
            fastcgi_pass   unix:/var/run/nginx-fcgi.sock;
        }
```
重启nginx，访问地址http://192.168.216.145/cgi-bin/graph.cgi

看到已经有图了
![Alt text](/images/evoup/rrdtool_load_graph2.png)







###参考链接
http://oss.oetiker.ch/rrdtool/doc/rrdcgi.en.html

http://wiki.qpsmtpd.org/doku.php?id=resources:statistics:rrdcgi-sample
