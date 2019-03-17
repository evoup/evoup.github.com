---
layout: post
title: "linux下模拟df.c源码"
date: 2013-10-16 18:09
comments: true
categories:              [c-language,monitor]
---
主要实现df的基本不带参数的功能，连界面都不一样，凑活用，见代码：

<!-- more -->

```c
#include <stdio.h>
#include <stdlib.h>
#include <mntent.h>
#include <sys/vfs.h>

int main(void)
{
 struct mntent *ent;
 FILE *aFile;

 aFile = setmntent("/etc/mtab", "r");
 if (aFile == NULL) {
   perror("setmntent");
   exit(1);
 }
 struct statfs diskInfo;
 unsigned long long blocksize;
 unsigned long long totalsize;
 unsigned long long freeDisk;
 unsigned long long availsize;
 unsigned long long used;
 while (NULL != (ent = getmntent(aFile))) { //获取各挂载点的信息
   printf("=========================================================================\
==================================================\n");
   //根据挂载点，确认磁盘空间
   statfs(ent->mnt_dir,&diskInfo);
   blocksize = diskInfo.f_bsize; //每个block里面包含的字节数
   totalsize = blocksize * diskInfo.f_blocks; //总的字节数
   freeDisk = diskInfo.f_bfree*blocksize; //再计算下剩余的空间大小
   availsize = diskInfo.f_bavail*blocksize;
   //>10换算成KB
   used=totalsize-freeDisk;
   printf("FS == %s MOUNTPOINT == %s TOTAL_SIZE == %lu KB DISK_FREE == %ld KB USED ==\
%ld KB avail == %ld KB\n", ent->mnt_fsname, ent->mnt_dir,(int)(totalsize>>10),
(int)(freeDisk>>10),(int)(used>>10),(int)(availsize>>10));
 }
 endmntent(aFile);
   printf("=========================================================================\
==================================================\n");
}
```

附一张运行截图
![Alt text](/images/evoup/df.png)