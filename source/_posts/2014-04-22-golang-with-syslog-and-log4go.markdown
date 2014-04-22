---
layout: post
title: "golang使用syslog日志系统和安装使用第三方包log4go"
date: 2014-04-22 11:25
comments: true
categories: [golang] 
---

###使用syslog
首先介绍下golang的syslog的用法，因为这个其实也很好用，只不过比log4go缺点在于要用syslog那一套（如newsyslog）来手工设置syslog.conf来轮询日志。接下来是一个基本的用法，只有serverity的场景:
<!-- more -->

```go
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

或者使用shell调用的方式，理论上效率会低一点:
```go
/// using logger
cmd := exec.Command("logger", "-p", "local0.err", "-t", "bash", "hello bash")
err = cmd.Run()
if err != nil {  
    log.Fatal("error running cmd!")  
}
```

然后是一个可以log serverity和log facility都可以用上的例子:
```go
package main
import(
    "log/syslog"
	"fmt"
)
testLogger, err := syslog.New(syslog.LOG_WARNING|syslog.LOG_LOCAL0, "mytag")
defer testLogger.Close()
if err!=nil {
    fmt.Printf("New() failed: %v\n",err)
} else {
    fmt.Println("logger ok")
	test.Logger.Write([]byte("test go syslog"))
}
```

###log4go的安装
言归正传讲安装，不要被log4go的README搞晕了

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

###使用log4go
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

```go
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

代码就不多说了，唯一值得注意的是log4go貌似达到指定的日志数上限了之后就不会再去掉旧的文件，更新新的文件了，也就是再也无法记录日志，这个是需要程序考虑的，完。

