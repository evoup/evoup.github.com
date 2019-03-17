---
layout: post
title: "octopress2报错undefined method `[] for nill:NilClass"
date: 2014-02-09 22:32
comments: true
categories: octopress
---

今天在用octopress生成博客静态页的时候，出现如下警告
` Liquid Exception: undefined method '[]' for nil:NilClass `
看下错误上下文先，上次记得这个问题是括号不能连写导致的，这次又是为啥呢？

<!-- more -->
```
$ rake generate
## Generating Site with Jekyll
unchanged sass/screen.scss
Configuration from E:/octopress2/evoup.github.com/_config.yml
Building site: source -> public
## Generating Categories..
 => Creating Categories Tag Cloud
 => Creating Categories Sidebar
Liquid Exception: undefined method `[]' for nil:NilClass in 2014-02-08-cocos2d-x
-cocos2d-iphone-cocos2d-html5-sprite-and-animation-summary.markdown
E:/octopress2/evoup.github.com/plugins/pygments_code.rb:14:in `highlight'
E:/octopress2/evoup.github.com/plugins/code_block.rb:82:in `render'
c:/Ruby193/lib/ruby/gems/1.9.1/gems/liquid-2.3.0/lib/liquid/block.rb:94:in `bloc
k in render_all'
c:/Ruby193/lib/ruby/gems/1.9.1/gems/liquid-2.3.0/lib/liquid/block.rb:92:in `coll
ect'
c:/Ruby193/lib/ruby/gems/1.9.1/gems/liquid-2.3.0/lib/liquid/block.rb:92:in `rend
er_all'
c:/Ruby193/lib/ruby/gems/1.9.1/gems/liquid-2.3.0/lib/liquid/block.rb:82:in `rend
```
是看来是语法高亮的语法问题，认为代码有错，2个括号居然不能分开写：

> sprite->setPosition( ccp(100, 240) ); //设置坐标为x=100,y=240
> <br/>
> 必须写成sprite->setPosition(ccp(100, 240)); //设置坐标为x=100,y=240

以下问题发现在这个ruby版本
ruby 1.9.3p484 (2013-11-22) [i386-mingw32]

但奇怪的是公司里的环境没有问题，先记一笔备忘。

