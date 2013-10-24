---
layout: post
title: "编译rebar项目中单个文件的vim脚本"
date: 2013-10-24 12:41
comments: true
categories: [erlang,vim] 
---
本人用rebar写程序总免不了rebar complie和rebar generate一番，项目关联比较多整个过程非常慢，这是很令人沮丧的。有没有什么方法能像C语言的makefile一样，只编译需要的文件呢？答案是没有现成的，就我所知rebar没有这么高端的功能去判断哪些文件是变化过的。那就通过其他方式提高生产效率，想到了erlc，单个文件编译是没有什么问题。
<!-- more -->
方法也很EZ
```bash
erlc -o ./ebin foo.erl
```
###小试牛刀
于是我如法炮制，初步写出了如下vim脚本。（我的是cshell，如果是bash请把对应的set变量部分改成bash的风格）
```bash
"编译当前erlang文件且部署到rebar项目的rel目录的对应位置
map bd :call CompileErlDeploy()<CR>
func! CompileErlDeploy()
  exec "w"
  exec "!erlc -o /tmp %"
  exec "!set bar = \"`ls apps/`\""
  exec "!mv /tmp/`basename %<.beam` rel/`ls apps/`/lib/`ls apps/`-`grep vsn apps/`ls apps/`/src/`ls apps/`.app.src | sed 's/{vsn, \"//g' | sed 's/\"},//g'`/ebin/"
  exec "!`pwd`/rel/`ls apps/`/bin/`ls apps/` stop"
endfunc

map bf :call RunErl()<CR>
func! RunErl()
  exec "!`pwd`/rel/`ls apps/`/bin/`ls apps/` start"
endfunct

```
基本思路是先找到当前文件所在的路径，然后使用erlang的独立编译器erlc直接编译出该文件的beam字节码，最后拷贝到部署目录rel所在的位置，当然部署是有版本的，版本的信息到apps目录下的.app.src的vsn中提取。

###进一步优化
对CompileErlDeploy合并有：
```bash
"编译当前erlang文件且部署到rebar项目的rel目录的对应位置
map bd :call CompileErlDeploy()<CR>
func! CompileErlDeploy()
  exec "w"
  exec "!erlc -o /tmp % && set bar = \"`ls apps/`\" && mv /tmp/`basename %<.beam` rel/`ls apps/`/lib/`ls apps/`-`grep vsn apps/{$bar}/src/{$bar}.app.src | sed 's/{vsn, \"//g' | sed 's/\"},//g'`/ebin/ && `pwd`/rel/`ls apps/`/bin/`ls apps/` stop"
endfunc

map bf :call RunErl()<CR>
func! RunErl()
  exec "!`pwd`/rel/`ls apps/`/bin/`ls apps/` start"
endfunct
```
###使用的方法
进入rebar项目的根目录，sudo vim apps/项目名/src/源码.erl
随后使用bd即可编译出源码.beam,并且移到rel目录下beam应该的位置,而bf则可以启动该rebar应用程序。
需要注意的是，至少要rebar generate成功生成一次rel目录的文件，否则替换啥呢：）

###参考链接
<a href="http://www.ibm.com/developerworks/cn/linux/l-vim-script-1/">http://www.ibm.com/developerworks/cn/linux/l-vim-script-1/</a>
