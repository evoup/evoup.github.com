---
layout: post
title: "使用expect脚本检测telnet瞬断"
date: 2014-01-08 18:17
comments: true
categories: [shell,monitor]
---

分享一个检查telnet瞬断的脚本，我用调用它来检测thrift的TCP9090端口一连上就断开连接的状态。

直接上代码了
<!-- more -->

```sh
#!/usr/local/bin/expect
spawn telnet 127.0.0.1 9090 
set timeout  1
expect {
        "*Escape character is*" {
                exp_continue
        }
        "*Connection closed by foreign host*" {
                send "note: 1) detect a unsafe connection"
        }
}
expect {
        "*Connection closed by foreign host*" {
                send "note: 2) detect a unsafe connection"
        }
        send "\\003"
        exit
}
expect "*Connection closed by foreign host*"
send "note: 3) disconnect by client cause timeout or not immediately exit expect"
exit
expect eof

exit
expect eof

```

调用者程序可以根据expect输出的note: 3) disconnect by client cause timeout or not immediately exit expect得知telnet上去瞬间失去连接。