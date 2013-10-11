---
layout: post
title: "freebsd下如何创建和使用Ctag"
date: 2013-10-11 14:48
comments: true
categories:                  vim
---
以php语言为例
```bash
find . -name "*.php" -exec exctags --language-force=php {} +
```
在freebsd里面，我还是去http://ctags.sourceforge.net/下载编译安装了ctags，老实说freebsd自带的ctags一点也不好用，除了不能直接ctags -R *以外，据说vim的taglist还不支持这个语法，还是要去下载的这个版本。但是和freebsd默认的ctags就冲突了
于是
```bash
pwd
/usr/bin/
mv ctags ctags.orig
ln -s /usr/local/bin/ctags ctags
```

对于c语言可能包含的h文件
```bash
find . -name "*.*" -exec exctags {} +
```

题外话，要将find到的文件移动到其他目录，可以这么干
find . -name "*.h" -exec mv {} temp/ \;

彩色查找
```bash
find . ! -path "*.svn*" -type f -exec grep -n --color CM_MISC {} \;
```

替换全部代码中的abc为123，排除svn文件夹
```bash
find . ! -path "*.svn*" -type f -exec sed -i "s/abc/123/g" {} \;
```
