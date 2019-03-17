---
layout: post
title: "用telnet透过smtp协议发送邮件"
date: 2012-12-19 15:40
comments: true
categories:  shell
---

调试程序的时候需要了解SMTP的工作原理，由于这是应用层文本的TCP协议，所以直接通过telnet调试即可。

<!-- more -->

```sh
$ telnet mail.madhouse-inc.com 25
Trying 172.16.27.3...
Connected to mail.madhouse-inc.com.
Escape character is '^]'.
220 mail2.madhouse-inc.com ESMTP Mail System
EHLO mail.madhouse-inc.com
250-mail.madhouse-inc.com
250-PIPELINING
250-SIZE 512000000
250-VRFY
250-ETRN
250-STARTTLS
250-AUTH LOGIN PLAIN
250-AUTH=LOGIN PLAIN
250-ENHANCEDSTATUSCODES
250-8BITMIME
250 DSN
250
502 5.5.2 Error: command not recognized
AUTH LOGIN
334 VXNlcm5hbWU6
用户名的base64字符串
334 UGFzc3dvcmQ6
密码的base64字符串
235 2.0.0 Authentication successful
MAIL FROM:<test@163.com>
250 2.1.0 Ok
RCPT TO:<evoup@gmail.com>
250 2.1.5 Ok
DATA
354 End data with <CR><LF>.<CR><LF>
test now!
.
250 2.0.0 Ok: queued as 09AE022931
NOOP
250 2.0.0 Ok
```