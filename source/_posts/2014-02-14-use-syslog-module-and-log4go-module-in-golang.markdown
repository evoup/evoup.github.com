---
layout: post
title: "use syslog module and log4go module in golang"
date: 2014-02-14 18:23
comments: true
categories: 
---

golang使用syslog日志系统和安装使用第三方包log4go


首先介绍下golang的syslog的用法，因为这个其实也很好用，只不过比log4go缺点在于要用syslog那一套（如newsyslog）来轮询日志。
只不过缺憾的是用API只能进行log facility的设置,不能进行log level的设置

```
// syslog init
func LogInit() *log.Logger {
    Log, err := syslog.NewLogger(syslog.LOG_DEBUG, log.Lmicroseconds) //设置log facility为LOG_DEBUG
    if err != nil {
        log.Fatal(err)
    }
    return Log
}
//然后打印日志的时候可以调用对应的log level
```

或者使用shell调用的方式，理论上效率会低一点
```
/// using logger
cmd := exec.Command("logger", "-p", "local0.err", "-t", "bash", "hello bash")
err = cmd.Run()
if err != nil {  
    log.Fatal("error running cmd!")  
}
```

既然是如此的不好用，为什么不用下可以自动
言归正传讲安装，不要被下面这个README搞晕了

<!-- more -->

```
Please see http://log4go.googlecode.com/

Installation:
- Run `goinstall log4go.googlecode.com/hg`

Usage:
- Add the following import:
import l4g "log4go.googlecode.com/hg"

Acknowledgements:
- pomack
  For providing awesome patches to bring log4go up to the latest Go spec

```

其实正确的安装方法如下：

```sh
go get code.google.com/p/log4go
```

安装完成后默认路径位于
```sh
/usr/local/go/src/pkg/code.google.com/p/log4go
```


打开文档看怎么用，有2种方法：
####第一种
在安装好log4go的机器上运行
```sh
godoc -http=:6060
```

然后用浏览器查看
http://host:6060/src/pkg/code.google.com/p/log4go/examples/

####第二种
直接查看该目录下的例子
/usr/local/go/src/pkg/code.google.com/p/log4go/examples

主要来看一个日志轮询的怎么做

```
package main

import (
	"bufio"
	"fmt"
	"io"
	"os"
	"time"
)

import l4g "code.google.com/p/log4go"

const (
	filename = "flw.log"
)

func main() {
	// Get a new logger instance
	log := l4g.NewLogger() //实例化日志对象

	// Create a default logger that is logging messages of FINE or higher
	log.AddFilter("file", l4g.FINE, l4g.NewFileLogWriter(filename, false))
	log.Close()

	/* Can also specify manually via the following: (these are the defaults) */
	flw := l4g.NewFileLogWriter(filename, false)
	flw.SetFormat("[%D %T] [%L] (%S) %M")
	flw.SetRotate(false)
	flw.SetRotateSize(0)
	flw.SetRotateLines(0)
	flw.SetRotateDaily(false)
	log.AddFilter("file", l4g.FINE, flw)

	// Log some experimental messages
	log.Finest("Everything is created now (notice that I will not be printing to the file)")
	log.Info("The time is now: %s", time.Now().Format("15:04:05 MST 2006/01/02"))
	log.Critical("Time to close out!")

	// Close the log
	log.Close()

	// Print what was logged to the file (yes, I know I'm skipping error checking)
	fd, _ := os.Open(filename)
	in := bufio.NewReader(fd)
	fmt.Print("Messages logged to file were: (line numbers not included)\n")
	for lineno := 1; ; lineno++ {
		line, err := in.ReadString('\n')
		if err == io.EOF {
			break
		}
		fmt.Printf("%3d:\t%s", lineno, line)
	}
	fd.Close()

	// Remove the file so it's not lying around
	os.Remove(filename)
}
```

