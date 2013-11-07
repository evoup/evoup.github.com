---
layout: post
title: "vim在sudo下的使用当前用户的配置文件"
date: 2011-11-06 15:15
comments: true
categories:                 vim
---

部门配备了freebsd做为统一的开发环境，大家登上各自的jail进行软件开发任务。但是发现执行sudo vim之后，居然使用的是root的vim配置文件。

那么有什么办法能够使用自己的配置文件呢？其实再简单不过,使用-E参数
```
       -E  The -E (preserve environment) option will override the env_reset
           option in sudoers(5)).  It is only available when either the match-
           ing command has the SETENV tag or the setenv option is set in sudo-
           ers(5).
```

```bash
sudo -E vim file
```
然后就可以使用自己的配置文件了。
