---
layout: post
title: "git免密码使用https协议"
date: 2013-12-30 22:09
comments: true
categories: version_control 
---

git支持的协议主要有git、ssh、http、https这几种，国内防火墙对github的作用下，导致长期使用github服务的程序开发人员要忍受缓慢的push和pull速度。现在终于找到了一个比较不错的方法提升工作效率，那就是使用goagent+git/https的方式。
<!-- more -->

goagent配置这里就不提了，主要给出git端的设置参考,假设是在bash终端下操作git，需要设置这2个环境变量，写到~/.bashrc中再source一下即可

```bash
export https_proxy="127.0.0.1:8087"
export http_proxy="127.0.0.1:8087"
```

其实这里是通过linux的http/https代理设置的方法来加速！
如果是tcsh

```bash
setenv https_proxy "127.0.0.1:8087"
setenv http_proxy "127.0.0.1:8087"
```

同样是把以上的代码加到~/.cshrc中再source一下

速度的问题解决了之后，你会发现其实每次都非常讨厌输入用户名和密码，那请看下文

执行

```bash
touch $HOME/.git-credentials
```

编辑$HOME/.get-credentials
加入

```bash
https://username:password@github.com
```

其中username和password分别为你的git用户名和密码

再执行

```bash
git config --global credential.helper store
```

这样git push就不需要输入密码了，享受成就感吧


