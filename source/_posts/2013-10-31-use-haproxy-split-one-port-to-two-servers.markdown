---
layout: post
title: "用haproxy分1个端口到多个服务器应用"
date: 2013-10-31 10:57
comments: true
categories: haproxy
---
今天实验了一下鼎鼎大名号称七层负载均衡的haproxy软件，果然是强得五体投地。我只有一个

haproxy的安装
wget http://haproxy.1wt.eu/download/1.4/src/haproxy-1.4.24.tar.gz
tar xzf haproxy-1.4.24.tar.gz
cd haproxy-1.4.24
gmake TARGET=freebsd
sudo gmake install
```bash

global
    maxconn 5120
    chroot /usr/local/haproxy14
    daemon
    quiet
    nbproc 2
    pidfile /usr/local/haproxy14/run/haproxy.pid

defaults
    timeout connect 5s
    timeout client 50s
    timeout server 20s
    log 127.0.0.1 local3 err #日志级别[err warning info debug]


listen http
    bind :8080
    timeout client 1h
    tcp-request inspect-delay 2s
    acl is_http req_proto_http
    tcp-request content accept if is_http
    server server-http :8081
    use_backend tcpserver if !is_http

backend tcpserver
    mode tcp
    timeout server 1h
    server server-tcp :13081
```

实现的功能是对8080端口的socket请求，如果判断协议为http则路由到8081，如果为其他未知的tcp协议则路由到13081所在的socket服务器端口。

###参考链接

http://cbonte.github.io/haproxy-dconv/configuration-1.4.html
