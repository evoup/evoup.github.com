---
layout: post
title: "64位freebsd8.1下安装ffmpeg和php扩展"
date: 2014-02-12 17:27
comments: true
categories: [freebsd,php] 
---


帮同事解决一个开源音视频转换处理软件ffmpeg在freebsd8.1下安装的问题，我也是装了2个下午，绕了个圈子下载了个没用的扩展浪费时间。

<!-- more -->

###环境freebsd8.1 amd64
我拿个了干净的系统做试验，安装的时候选择了develop模式和customize，customize里只选择了ports,然后php是装在了/usr/local/php53目录下。

###安装过程描述
####ffmpeg的安装
主要采用pkg_add ffmpeg.tbz的方式来安装。
然后看报错的依赖缺少啥一个一个装上去，途中需要的包可以通过下面的命令来获取：

```sh
fetch ftp://ftp.fi.freebsd.org/pub/FreeBSD/ports/amd64/packages-8.1-release/Latest/ffmpeg.tbz
fetch ftp://ftp.fi.freebsd.org/pub/FreeBSD/ports/amd64/packages-8.1-release/Latest/x264.tbz
```

采用pkg_add快速安装包
安装的时候反正就是不断用下面的语句试探

```sh
sudo pkg_add ffmpeg.tbz
```

用pkg_add一个一个装，直到ffmpeg安装完成。



如果你也是这么装，那么全部东西下完，大概有这么多依赖OMG

```sh
>ls
damageproto.tbz  gpac-libgpac.tbz  libXau.tbz      libdrm.tbz            libvorbis.tbz   png.tbz               xvid.tbz
dri2proto.tbz    jpeg.tbz          libXdamage.tbz  libiconv.tbz          libxcb.tbz      schroedinger.tbz
expat.tbz        kbproto.tbz       libXdmcp.tbz    libogg.tbz            libxml.tbz      x264.tbz
faad2.tbz        libGL.tbz         libXext.tbz     libpthread-stubs.tbz  libxml2.tbz     xextproto.tbz
ffmpeg.tbz       libGLU.tbz        libXfixes.tbz   libtheora.tbz         orc.tbz         xf86vidmodeproto.tbz
fixesproto.tbz   libX11.tbz        libXxf86vm.tbz  libtool.tbz           pkg-config.tbz  xproto.tbz
```

嫌麻烦的可以先下好软件然后批量装

```sh
find . -name "*.tbz" -exec pkg_add {} \;
```

当然也免不了部分手动来部分包，窃喜~


----------------------------

####ffmpeg-php的安装
装php，./configure的时候出现出错，查看config.log

` /usr/bin/ld: cannot find -liconv `

需要再用pkg_add安装libxml，重新安装libiconv，用port安装，谈到要安装libtool213,再用pkg_add去装，卸掉libiconv，重新用port安装，下载php5.3编译通过，再去下载ffmpeg-php，用phpize的方式编译成动态库，会报告少autoconf，此时在到port里安装autoconf213，会让安装perl，等待安装完成就是。


下载正确的扩展,不要用sf那个2005年的扩展，不能用。

```sh
fetch http://downloads.sourceforge.net/project/ffmpeg-php/ffmpeg-php/0.6.0/ffmpeg-php-0.6.0.tbz2
tar xjf ffmpeg-php-0.6.0.tbz2
cd ffmpeg-php
phpize
```

在phpize的时候可能出现

```sh
[evoup@freebsd81amd64 ffmpeg]>/usr/local/php53/bin/phpize
Configuring for:
PHP Api Version:         20090626
Zend Module Api No:      20090626
Zend Extension Api No:   220090626
Cannot find autoconf. Please check your autoconf installation and the
$PHP_AUTOCONF environment variable. Then, rerun this script.
```

此时需要在环境变量中指定好autoconf和autoheader的路径，我是加载.cshrc文件中

```sh
setenv PHP_AUTOCONF "/usr/local/bin/autoconf"
setenv PHP_AUTOHEADER "/usr/local/bin/autoheader"
```

完事之后不要忘记重载~/.cshrc

```sh
source ~/.cshrc
```

phpize完了之后可以开始configure了

```sh
./configure --with-php-cofig=/usr/local/php53/bin/php-config
/usr/local/include/ffmpeg/avcodec.h:30:30: error: libavutil/avutil.h: No such file or directory
```

其实这个是ffmpeg安装好应该有的库

```sh
$ locate avutil.h
/usr/home/evoup/software/libavutil/avutil.h
/usr/local/include/ffmpeg/avutil.h
/usr/local/include/libavutil/avutil.h
```

怎么装上去？嘿嘿，直接把相关的路径做软连接

```sh
sudo ln -s /usr/local/include/libavcodec/ /usr/include/libavcodec/
sudo ln -s /usr/local/include/libavcodec/ /usr/include/libavcodec
sudo ln -s /usr/local/include/libavdevice/ /usr/include/libavdevice
sudo ln -s /usr/local/include/libavfilter/ /usr/include/libavfilter
sudo ln -s /usr/local/include/libavformat/ /usr/include/libavformat
sudo ln -s /usr/local/include/libavutil/ /usr/include/libavutil
```

然后再次

```sh
make 
sudo make install
Installing shared extensions:     /usr/local/php53/lib/php/extensions/no-debug-non-zts-20090626/
```

安装ok,接下来加载到php,要做的是看下php的ini位置，放进去就是

```sh
$ /usr/local/php53/bin/php -i | grep Conf
Configure Command =>  './configure'  '--prefix=/usr/local/php53'
Configuration File (php.ini) Path => /usr/local/php53/lib
Loaded Configuration File => /usr/local/php53/lib/php.ini
Configuration
```

说明在/usr/local/php53/lib下，于是把编译好的so移到该目录下

```sh
$ sudo mv /usr/local/php53/lib/php/extensions/no-debug-non-zts-20090626/ffmpeg.so /usr/local/php53/lib
```

最后在php.ini中指定

```sh
extension=ffmpeg.so
```

-----------------------

观察劳动成果

```sh
$ /usr/local/php53/bin/php -i | grep ffmpeg
ffmpeg
ffmpeg-php version => 0.6.0-svn
ffmpeg-php built on => Feb 12 2014 17:10:47
ffmpeg-php gd support  => disabled
ffmpeg libavcodec version => Lavc52.20.1
ffmpeg libavformat version => Lavf52.31.0
ffmpeg swscaler => disabled
ffmpeg.allow_persistent => 0 => 0
ffmpeg.show_warnings => 0 => 0
```

没问题收工。



参考互联网文章《编译FFMpeg和FFMpeg-php》linux版的
