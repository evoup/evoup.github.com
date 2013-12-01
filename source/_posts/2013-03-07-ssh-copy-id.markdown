---
layout: post
title: "非常便捷的ssh信任制作脚本ssh-copy-id"
date: 2013-10-17 17:29
comments: true
categories:           shell
---

手册一把原来这个工具就是shell脚本，ssh-copy-id这个工具可以方便的把公钥直接拷贝到对方服务器信任列表。比起来回切换服务器拷贝公钥方便多了。

<!-- more -->

实验：hostA: 172.16.30.184:22  hostB:211.136.104.189:9999
```bash
>ssh-keygen -t rsa

Generating public/private rsa key pair.

Enter file in which to save the key (/home/user/.ssh/id_rsa):

/home/user/.ssh/id_rsa already exists.

Overwrite (y/n)? y

Enter passphrase (empty for no passphrase):

Enter same passphrase again:

Your identification has been saved in /home/user/.ssh/id_rsa.

Your public key has been saved in /home/user/.ssh/id_rsa.pub.

The key fingerprint is:

74:59:16:2c:01:0c:2a:21:b4:49:8d:0e:51:2f:d9:a4 user@user-arch.madhouse.cn

The key's randomart image is:

+--[ RSA 2048]----+

|+== . .o...o+.   |

|oooO .  . .+.    |

|ooE +   . o.     |

| . o   . .       |

|        S        |

|                 |

|                 |

|                 |

|                 |

+-----------------+
```
然后运行如下脚本，按照提示输入密码即可
```bash
ssh-copy-id '-p 9999 -i /usr/home/user/.ssh/id_rsa.pub user@211.136.104.189'
```
更加简单的使用方法，不带i参数，默认使用当前用户的id_rsa.pub公钥
```bash
ssh-copy-id user@211.136.104.189
```
这样子就可以了。

如果要建立非22端口的信任。
例：建立到211.136.221.207的信任，ssh的端口为8887，用户名为user
```bash
ssh-copy-id "-p8887 user@211.136.221.207"
```

参考：

http://mikegerwitz.com/ssh-copy-id-and-sshd-port/

http://blogread.cn/it/article/6103

