---
layout: post
title: "非常便捷的ssh信任制作脚本ssh-copy-id"
date: 2013-10-17 17:29
comments: true
categories: 
---

手册一把原来这个工具就是shell脚本，ssh-copy-id这个工具可以方便的把公钥直接拷贝到对方服务器信任列表。比起来回切换服务器拷贝公钥方便多了。

<!-- more -->

实验：hostA: 172.16.30.184:22  hostB:211.136.104.189:9999
```bash
>ssh-keygen -t rsa

Generating public/private rsa key pair.

Enter file in which to save the key (/home/yin/.ssh/id_rsa):

/home/yin/.ssh/id_rsa already exists.

Overwrite (y/n)? y

Enter passphrase (empty for no passphrase):

Enter same passphrase again:

Your identification has been saved in /home/yin/.ssh/id_rsa.

Your public key has been saved in /home/yin/.ssh/id_rsa.pub.

The key fingerprint is:

74:59:16:2c:01:0c:2a:21:b4:49:8d:0e:51:2f:d9:a4 yin@yin-arch.madhouse.cn

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
ssh-copy-id '-p 9999 -i /usr/home/yin/.ssh/id_rsa.pub yin@211.136.104.189'
```

参考：
http://mikegerwitz.com/ssh-copy-id-and-sshd-port/
http://blogread.cn/it/article/6103