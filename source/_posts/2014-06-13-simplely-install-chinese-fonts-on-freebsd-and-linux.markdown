---
layout: post
title: "freebsd和linux上安装中文字体"
date: 2014-06-13 17:45
comments: true
categories: [freebsd,rrdtool,linux]
---

###使用中文字体及其安装
首先说明，因为需要在不同的平台下让rrdtool显示的图片看起来舒服点，考虑安装中文字体。这里搜罗下方法。（这种方式仅仅是为了程序工作正常，需要使自己的KDE或者GNOME系统中文显示正常可以不用看本文了。）
<!-- more -->

###freebsd下
直接把字体文件放到/usr/local/share/fonts下，然后刷新字体缓存

```bash
sudo cp ukai.ttf /usr/local/share/fonts/
sudo fc-cache -f -v
```

###centos下
把字体放到/usr/share/fonts/msfonts下，然后执行几个命令

```bash
sudo mkdir /use/share/fonts/msfonts
sudo cp ukai.ttf /usr/share/fonts/msfonts/
cd /usr/share/fonts/msfonts/
sudo mkfontscale
sudo mkfontdir
sudo fc-cache -fv
```

更EZ的方式有

```bash
sudo yum  –y  install  fonts-chinese
```

于是就可以在rrdtool里指定字体为ukai.ttf了，是不是各种简单:)
