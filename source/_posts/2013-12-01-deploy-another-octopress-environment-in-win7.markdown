---
layout: post
title: "克隆你的octopress博客到win7"
date: 2013-12-01 18:25
comments: true
categories:  octopress
---
由于之前octopress博客是在公司的机器上搭的，想到在家里笔记本上的win7也搞一个环境，方便及时写写技术博客。参考网文《【像黑客一样写博客之五】博客克隆》进行摸索，终于实践成功。

但我的方法稍微有点区别,这里采用clone的方式比较快，主要省去一个remote add环节：）只要主要注意几个问题，就可以轻松搞定，顺序看就行。

<!-- more -->
##环境篇：
这个可以参考大多数搭建octopress博客的文章了，这里一笔带过，之把出现的问题记一笔方便日后查阅。

###问题1（ruby的版本）

当我装到bundle install的时候，出现了如下问题
```
An error occurred while installing rdiscount (2.0.7.3), and Bundler cannot
continue.
```
查阅README.markdown
**Note**: Octopress requires a minimum Ruby version of `1.9.3-p0`.
原来需要安装ruby1.9.3。


###问题2 （在装完devkit之前utf-8环境变量不要设置）

而安装ruby1.9.3,相应需要安装DevKit-tdm-32-4.5.2-20111229-1559-sfx.exe
直接解压到e:\devkit
然后再
```
cd e:\devkit
ruby dk.rb init
C:/Ruby193/lib/ruby/1.9.1/win32/registry.rb:173:in `tr': invalid byte sequence in UTF-8 (ArgumentError)
```
原来是环境变量不能先设置为UTF-8，这个是要特别注意的。

然后就可以执行devkit的安装了：
```
ruby dk.rb init
ruby dk.rb install
```
接着设置环境变量（win7可以用开始搜索程序和文件，输入编辑系统环境变量）：
```
LANG=zh_CN.UTF-8
LC_ALL=zh_CN.UTF-8
```

继续安装2个必要的gem
```
gem install rdoc bundler
bundle install
```
这样开发环境就算和之前一致了，这里没有说git的安装，和网上说的没有特别之处，反正装完了需要把git/bin目录加到环境变量PATH中去。


##博客恢复篇
签出自己的octopress项目,我的放在e:\octopress
```
cd e:\octopress\
git clone https://github.com/you/you.github.com.git
git config --global user.email "you@example.com"
git config --global user.name "Your Name"
cd you.github.com
rake setup_github_pages
cd _deploy
git checkout source
git pull origin master
```
所有的资料都回来了！
然后切记切换回source分支，因为master分支的是octopress通过rake部署自动提交进行维护的。
```
git checkout source
```
再次rake generate和rake preview以及rake deploy发现已经可以发布了，最后还需要把相关的提交了，git add . 和git commit -a以及git push origin source

最后注意游走在不同的octopress博客环境处理博客之前，需要同步github仓库的数据
```
cd e:\octopress\you.github.com 
git pull origin source 
cd _deploy  
git pull origin master  
```
