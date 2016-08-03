---
layout: post
title: "freebsd下如何使用ctags为项目生成tag文件"
date: 2010-07-23 14:48
comments: true
categories: [freebsd,vim,ctags]
---
用vim写代码免不了使用ctags，但是在freebsd下稍稍有点区别，你需要Exuberant Ctags，它是ctags的扩展实现，比freebsd自带的ctags功能更加强大，老实说freebsd自带的ctags一点也不好用，除了不能直接ctags -R *以外，据说vim的taglist还不支持这个语法。

<!-- more -->

在freebsd里面，可以去ctags.sourceforge.net下载编译安装了ctags，还是要去下载的这个版本。但是和freebsd默认的ctags就冲突了。

其实可以直接port安装了,装完之后这个软件其实叫做exctags。

```bash
whereis ctags
/usr/ports/devel/ctags
cd /usr/ports/devel/ctags
sudo make install clean
```

以php语言为例

```bash
find . -name "*.php" -exec exctags --language-force=php {} +
```

对于c语言可能包含的h文件的方法，我采用如下的方式：

```bash
find . -regex ".*\.\(c\|h\)" -exec exctags {} +
```

如果没有限制想把全部源码递归生成到一个tag文件则

```bash
cd /root/repos/httpd && exctags -R　*
```
