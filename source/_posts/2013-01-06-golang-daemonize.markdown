---
layout: post
title: "golang的守护进程化"
date: 2013-01-06 17:37
comments: true
categories: golang 
---
golang的守护进程化代码，参考了网上的代码，结合自己的项目，去掉了一些没用的代码，增加了fork时候的判断处理，我的机器是freebsd，fork会报errno:0,忽略，原因见代码注释。

<!-- more -->

```go
func Daemonize(nochdir, noclose int) error {
   var hLog *log.Logger
   hLog=LogInit();
   var ret, ret2 uintptr
   var err error
   Log( hLog, fmt.Sprintf("common.Daemonize][current ppid:%d", syscall.Getppid()) )

   //already a daemon
   if syscall.Getppid() == 1 {
       return nil
   }
   Log(hLog,"common.Daemonize][will daemonize")

   //fork off the parent process
   ret, ret2, err = syscall.RawSyscall(syscall.SYS_FORK, 0, 0, 0)
   if err != nil {
       var s string
       s=fmt.Sprintf("%T",err)
       if s != "syscall.Errno" {
           Log(hLog,"common.Daemonize][fork err:"+s)
               return err
       } else {
           Log(hLog,"common.Daemonize][fork no err:"+err.Error())
           //no problem see http://www.ibm.com/developerworks/aix/library/au-errnovariable/
       }
   }

   //failure
   if ret2 < 0 {
       os.Exit(-1)
   }

   //if we got a good PID, then we call exit the parent process.
   if ret > 0 {
       os.Exit(0)
   }
   Log(hLog,"common.Daemonize][forked,we in forked process")

   /* Change the file mode mask */
   _ = syscall.Umask(0)
   Log(hLog,"common.Daemonize][umask zero")

   //create a new SID for the child process
   s_ret, s_errno := syscall.Setsid()
   if s_ret < 0 || s_errno != nil {
       log.Printf("common.Daemonize][Error: syscall.Setsid errno: %d", s_errno)
   }
   Log(hLog,"common.Daemonize][sid seted")

   if nochdir == 0 {
       os.Chdir("/")
       Log(hLog,"common.Daemonize][chdir to root")
   }

   if noclose == 0 {
       f, e := os.OpenFile("/dev/null", os.O_RDWR, 0)
       if e == nil {
           fd := f.Fd()
           syscall.Dup2(int(fd), int(os.Stdin.Fd()))
           syscall.Dup2(int(fd), int(os.Stdout.Fd()))
           syscall.Dup2(int(fd), int(os.Stderr.Fd()))
           Log(hLog,"common.Daemonize][fs closed")
       }
   }

   Log(hLog,"common.Daemonize][daemonize done")
   return nil
}
```
