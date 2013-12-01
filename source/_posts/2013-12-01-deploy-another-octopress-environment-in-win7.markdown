---
layout: post
title: "克隆你的octopress博客到win7"
date: 2013-12-01 18:25
comments: true
categories:  octopress
---
由于之前octopress博客是在公司搭的，我想在家里也搞一个环境，方便及时写博客。参考互联网一博主的《【像黑客一样写博客之五】博客克隆》进行操作。

方法稍微有点区别,呵呵。归纳成了几个问题，顺序看就行。

<!-- more -->
##环境准备：
这个可以参考大多数搭建octopres博客的文章了，这里一笔带过，之把出现的问题记一笔方便日后查阅。

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

而安装ruby1.9.3需要先安装DevKit-tdm-32-4.5.2-20111229-1559-sfx.exe
直接解压到e:\devkit
然后再
```
cd e:\devkit
ruby dk.rb init
C:/Ruby193/lib/ruby/1.9.1/win32/registry.rb:173:in `tr': invalid byte sequence in UTF-8 (ArgumentError)
```
原来是环境变量不能先设置
```
LANG=zh_CN.UTF-8
LC_ALL=zh_CN.UTF-8
```
再次
```
ruby dk.rb init
ruby dk.rb install
```
再次设置环境变量为
```
LANG=zh_CN.UTF-8
LC_ALL=zh_CN.UTF-8
```
###问题3 （要安装bundle）
继续执行
```
gem install rdoc bundler
```
签出自己的octopress项目
```
git clone https://github.com/you/you.github.com.git
git config --global user.email "you@example.com"
git config --global user.name "Your Name"
rake generate
```
报错需要先bundle install
则先
```
bundle install
Your bundle is complete!
Use `bundle show [gemname]` to see where a bundled gem is installed.
```
安装ok

可以生成项目了
```
rake generate
```
进行本地预览
```
rake preview
```
然后访问 http://127.0.0.1:4000
查看blog，一切恢复正常。


###问题4 （找不到_deploy目录）
但是到了
```
rake deploy
```
又出现No such file or directory – deploy 
那么只要手工创建一个deploy文件夹 
再次运行rake deploy，但是发现虽然能够被部署，资料却没有被更新。
 我们还是删除_deploy文件夹 其实问题在于没有运行
 rake setup_github_pages
执行之后，发现多出_deploy
```
cd _deploy
git checkout source
git pull origin master
```
所有的资料都回来了！
然后切记切换会source
```
git checkout source
```
再次rake generate和rake preview以及rake deploy发现已经可以发布了，最后还需要把相关的提交了，git add . 和git commit -a以及git push origin source

最后注意游走在不同的octopress博客环境处理博客之前，需要同步github仓库的数据
```
cd Octopress/  
git pull origin source 
cd _deploy  
git pull origin master  
```
