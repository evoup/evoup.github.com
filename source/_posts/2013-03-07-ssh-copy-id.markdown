---
layout: post
title: "非常便捷的ssh信任制作脚本ssh-copy-id"
date: 2013-03-07 17:29
comments: true
categories:           shell
---

ssh-copy-id在版本比较高的freebsd的port中提供了，安装即可。手册一把原来这个工具就是shell脚本，ssh-copy-id这个工具可以方便的把公钥直接拷贝到对方服务器信任列表。比起来回切换服务器拷贝公钥方便多了。

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

这样就可以了。

###port中没有怎么办？
直接创建一个文件叫做/usr/local/bin/ssh-copy-id，注意文件权限要为555，以及所有者为root

```sh
#!/bin/sh

# Shell script to install your public key on a remote machine
# Takes the remote machine name as an argument.
# Obviously, the remote machine must accept password authentication,
# or one of the other keys in your ssh-agent, for this to work.

ID_FILE="${HOME}/.ssh/id_rsa.pub"

if [ "-i" = "$1" ]; then
  shift
  # check if we have 2 parameters left, if so the first is the new ID file
  if [ -n "$2" ]; then
    if expr "$1" : ".*\.pub" > /dev/null ; then
      ID_FILE="$1"
    else
      ID_FILE="$1.pub"
    fi
    shift         # and this should leave $1 as the target name
  fi
else
  if [ x$SSH_AUTH_SOCK != x ] && ssh-add -L >/dev/null 2>&1; then
    GET_ID="$GET_ID ssh-add -L"
  fi
fi

if [ -z "`eval $GET_ID`" ] && [ -r "${ID_FILE}" ] ; then
  GET_ID="cat ${ID_FILE}"
fi

if [ -z "`eval $GET_ID`" ]; then
  echo "$0: ERROR: No identities found" >&2
  exit 1
fi

if [ "$#" -lt 1 ] || [ "$1" = "-h" ] || [ "$1" = "--help" ]; then
  echo "Usage: $0 [-i [identity_file]] [user@]machine" >&2
  exit 1
fi

# strip any trailing colon
host=`echo $1 | sed 's/:$//'`

{ eval "$GET_ID" ; } | ssh $host "umask 077; test -d ~/.ssh || mkdir ~/.ssh ; cat >> ~/.ssh/authorized_keys" || exit 1

cat <<EOF
Now try logging into the machine, with "ssh '$host'", and check in:

  ~/.ssh/authorized_keys

to make sure we haven't added extra keys that you weren't expecting.

EOF
```

参考：

http://mikegerwitz.com/ssh-copy-id-and-sshd-port/

http://blogread.cn/it/article/6103

