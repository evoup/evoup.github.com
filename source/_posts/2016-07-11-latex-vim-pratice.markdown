---
layout: post
title: "linux下latex中文配置小记"
date: 2016-07-12 13:02
comments: true
categories: [latex]
---

好久没有更新blog了，又得开始啦，不然传不习乎都忘记了。这里主要记录下vim下配置latex的一些经过，整个过程还是比较快可以上手的。
这里没有去手动下载安装texlive2015，而是采取直接apt安装。
<!-- more -->
```
sudo apt-get install texlive texlive-math-extra texlive-latex-base texlive-latex-extra texlive-latex-recommended texlive-pictures texlive-science latex-beamer texlive-base texlive-bibtex-extra
```
当然也可以下载完整的latexlive-full，但比较占用磁盘空间。
```
sudo apt-get install texlive-full latex-beamer
```
由于磁盘空间充裕，为避免可能出现的各种麻烦，我选择了后者的完整安装。
安装完后，继续安装CJK的相关软件包（仅仅为了获取中文支持）：
```
sudo apt-get install latex-cjk-chinese ttf-arphic-* hbf-*
```
然后做测试，将下面的文件保存为helloworld.tex
```
\documentclass{article}
\usepackage{CJKutf8}
\begin{document}  
\begin{CJK}{UTF8}{gkai}
这是一个楷体中文测试，处理简体字。
\end{CJK}
\begin{CJK}{UTF8}{gbsn}
这是一个宋体中文测试，处理简体字。
\end{CJK}
\begin{CJK}{UTF8}{bkai}
這是一個big5編碼的楷體中文測試，處理繁體文字。  
\end{CJK}
\begin{CJK}{UTF8}{bsmi}
這是一個个big5編碼的明體中文測試，處理繁體文字。  
\end{CJK}  
\end{document}  
```
然后运行pdflatex helloworld.tex，正常的话应该就可以看到生成的能正确显示中文的pdf格式文件了。
安装evince，这是用来查看pdf文件的。
```
sudo apt-get install evince
```
查看pdf
```
evince helloworld.pdf
```
![Alt text](/images/evoup/evince_pdf_latex.png)

接下来就可以装字体了
```
sudo apt-get install fontforge
```
这样是不够的，还需要一堆windows的字体，宋体、仿宋、黑体等,这里提供下载：

<a target=_BLANK href="https://drive.google.com/open?id=0B_yrhfwhC4h2LUlYcDZkVWswQzQ">SimSun.ttc</a> 

<a target=_BLANK href="https://drive.google.com/open?id=0B_yrhfwhC4h2MXRZdDBZc3V4c1Eudo">simhei.ttf</a>

<a target=_BLANK href="https://drive.google.com/open?id=0B_yrhfwhC4h2anEwakdfdnBCQzg">simkai.ttf</a>
 
<a target=_BLANK href="https://drive.google.com/open?id=0B_yrhfwhC4h2eW9lczVrQ18zUUU">TimesNewRoman.ttf</a>
 
<a target=_BLANK href="https://drive.google.com/open?id=0B_yrhfwhC4h2UGVneTVFWkJ5dkE">STXingkai.ttf</a>

<a target=_BLANK href="https://drive.google.com/open?id=0B_yrhfwhC4h2LUlYcDZkVWswQzQ">simfang.ttf</a>
```
sudo cp simsun.ttc simhei.ttf simkai.ttf TimesNewRoman.ttf STXingkai.ttf simfang.ttf /usr/share/fonts
cd /usr/share/fonts
sudo chmod 644 simsun.ttc simhei.ttf simkai.ttf TimesNewRoman.ttf STXingkai.ttf simfang.ttf
sudo mkfontscale && sudo mkfontdir
sudo fc-cache -fsv
```
然后还是找不到某些字体，需要hack一下latexlive的配置文件
```
locate ctex-xecjk-winfonts.def | xargs sudo vim
修改其中的[SIMKAI.TTF]为KaiTi_GB2312
修改其中的[SIMFANG.TTF]为FangSong_GB2312
```
接下来就可以开始latex中文写作之旅了!

###后记
这里还没有考虑繁体中文的配置，以后补上。


###参考文章

Linux下Texlive的ctex包中文字体问题
https://huxuan.org/2012/07/14/chinese-font-problem-of-ctex-in-texlive-under-linux/

Ubuntu 14.04/14.10 系统安装 Latex及配置中文字体[修订]
http://blog.csdn.net/bensnake/article/details/43279329
