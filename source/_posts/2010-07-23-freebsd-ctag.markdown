---
layout: post
title: "freebsd下如何创建和使用Ctag"
date: 2010-07-23 14:48
comments: true
categories:                  vim
---
以php语言为例
```bash
find . -name "*.php" -exec exctags --language-force=php {} +
```

在freebsd里面，一开始我还是去ctags.sourceforge.net下载编译安装了ctags，老实说freebsd自带的ctags一点也不好用，除了不能直接ctags -R *以外，据说vim的taglist还不支持这个语法，还是要去下载的这个版本。但是和freebsd默认的ctags就冲突了！

于是就直接port安装了。
```bash
pwd
/usr/bin/
mv ctags ctags.orig
ln -s /usr/local/bin/ctags ctags
```

对于c语言可能包含的h文件的方法，我采用如下的方式：
```bash
find . -regex ".*\.\(c\|h\)" -exec exctags {} +
```

题外话，要将find到的文件移动到其他目录，可以这么干
find . -name "*.h" -exec mv {} temp/ \;

彩色查找CM_MISC关键字
```bash
find . ! -path "*.svn*" -type f -exec grep -n --color CM_MISC {} \;
```

替换全部代码中的abc为123，排除svn文件夹
```bash
find . ! -path "*.svn*" -type f -exec sed -i "s/abc/123/g" {} \;
```

我最后在~/.cshrc的配置文件中直接把查找关键字做成一个别名，如下：
```bash
alias f "find . \( -path '*.svn*' -o -path '*.fcgi*' -o -path '*tags*' -o -name '*.o' -o -name '*.so' -o -name '*.a' \) -p    rune -o -name .git -prune -o -print | xargs grep -in --color"
```
这样查找某个字符串的时候只需要输入如下指令即可
```bash
f '字符串'
```
