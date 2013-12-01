---
layout: post
title: "apache Mina 使用小记"
date: 2013-01-14 14:14
comments: true
categories:        [java]
---
apache mina使用小记 简介     Apache MINA是一个网络应用程序框架，用来帮助用户简单地开发高性能和高可靠性的网络应用程序。它提供了一个通过Java NIO在不同的传输例如TCP/IP和UDP/IP上抽象的事件驱动的异步API。 环境为jdk1.6+freebsd9（64bit）+apache-mina-2.0.7+vim 首先是环境变量的配置
```bash
setenv JAVA_HOME "/usr/local/diablo-jdk1.6.0/"
setenv JAVA_BIN "/usr/local/bin/"
```
下载mina

```bash
cd /usr/home/evoup/monsrvd-2.1/test/
fetch http://mirror.bjtu.edu.cn/apache/mina/mina/2.0.7/dist/apache-mina-2.0.7-bin.tar.gz
tar xzf apache-mina-2.0.7-bin.tar.gz
cp apache-mina-2.0.7/dist/mina-core-2.0.7.jar .
cp apache-mina-2.0.7/lib/slf4j-api-1.6.6.jar .
```
除此之外还需要slf4j这个是mina需要使用的日志库
```bash
fetch http://www.slf4j.org/dist/slf4j-1.7.2.tar.gz
tar xzf slf4j-1.7.2.tar.gz
cp slf4j-1.7.2/slf4j-api-1.7.2.jar .
cp slf4j-1.7.2/slf4j-nop-1.7.2.jar .
```
设置好JAVA的CLASSPATH
```bash
setenv CLASSPATH "/usr/home/evoup/project/management/monsrvd-2.1/test/slf4j-api-1.7.2.jar:/usr/home/evoup/project/management/monsrvd-2.1/test/slf4j-nop-1.7.2.jar:/usr/home/evoup/project/management/monsrvd-2.1/test/mina-core-2.0.7.jar:/usr/local/diablo-jdk1.6.0/lib:."
```
找到mina的TimeServer的例子 http://mina.apache.org/mina-project/userguide/ch2-basics/sample-tcp-server.html 代码最后是这个样子，先不用管细节


```java
import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.charset.Charset;
import org.apache.mina.core.service.IoAcceptor;
import org.apache.mina.core.session.IdleStatus;
import org.apache.mina.filter.codec.ProtocolCodecFilter;
import org.apache.mina.filter.codec.textline.TextLineCodecFactory;
import org.apache.mina.filter.logging.LoggingFilter;
import org.apache.mina.transport.socket.nio.NioSocketAcceptor;

import org.apache.mina.core.service.IoHandlerAdapter;
import org.apache.mina.core.session.IoSession;
import java.util.Date;
import org.slf4j.LoggerFactory;
import org.slf4j.Logger;

public class MinaTimeServer
{
   private static final int PORT = 9123;

   public static void main( String[] args ) throws IOException
   {
       IoAcceptor acceptor = new NioSocketAcceptor();

       acceptor.getFilterChain().addLast( "logger", new LoggingFilter() );
       acceptor.getFilterChain().addLast( "codec", new ProtocolCodecFilter( new TextLineCodecFactory( Charset.forName( "UTF-8" ))));

       acceptor.setHandler( new TimeServerHandler() );
       acceptor.getSessionConfig().setReadBufferSize( 2048 );
       acceptor.getSessionConfig().setIdleTime( IdleStatus.BOTH_IDLE, 10 );
       acceptor.bind( new InetSocketAddress(PORT) );
   }
}

class TimeServerHandler extends IoHandlerAdapter {
    static Logger logger = LoggerFactory.getLogger(TimeServerHandler.class);
   //static Logger logger = Logger.getLogger(TimeServerHandler.class);
   //异常处理
   public void exceptionCaught(IoSession session, Throwable cause) throws Exception {
       cause.printStackTrace();
   }
   //对接收到的数据进行业务处理，在这里我们不管收到什么信息都返回一个当前的日期
   public void messageReceived(IoSession session, Object message) throws Exception {
       String str = message.toString();
       if (str.trim().equalsIgnoreCase("quit")) {
           session.close(true);
           return;
       }
       logger.debug("Rec:" + str);
       Date date = new Date();
       session.write(date.toString());
       logger.debug("Message written...");
   }
   //当连接空闲时的处理
   public void sessionIdle(IoSession session, IdleStatus status) throws Exception {
       logger.debug("IDLE " + session.getIdleCount(status));
   }
}
```
```
[evoup@myhost]>telnet 127.0.0.1 9123
Trying 127.0.0.1...
Connected to localhost.

Escape character is '^]'.
Mon Jan 14 17:34:04 CST 2013
Mon Jan 14 17:34:04 CST 2013
```