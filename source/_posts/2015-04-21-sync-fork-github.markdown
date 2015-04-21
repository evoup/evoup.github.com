---
layout: post
title: "同步github分支过来的代码库"
date: 2015-04-21 13:53
comments: true
categories: [version_control]
---
主要的步骤是先查看当前fork的远程代码库
<!-- more -->
```
git remote -v
origin  https://github.com/YOUR_USERNAME/YOUR_FORK.git (fetch)
origin  https://github.com/YOUR_USERNAME/YOUR_FORK.git (push)
```

然后添加一个上游代码库，地址就是之前被fork的那个代码库
```
git remote add upstream https://github.com/ORIGINAL_OWNER/ORIGINAL_REPOSITORY
```


再次查看
```
git remote -v
origin    https://github.com/YOUR_USERNAME/YOUR_FORK.git (fetch)
origin    https://github.com/YOUR_USERNAME/YOUR_FORK.git (push)
upstream  https://github.com/ORIGINAL_OWNER/ORIGINAL_REPOSITORY.git (fetch)
upstream  https://github.com/ORIGINAL_OWNER/ORIGINAL_REPOSITORY.git (push)
```

接着用如下命令获取上游代码并合并到当前工作区
```
git fetch upstream
git checkout master
git merge upstream/master
```
最后git push origin master即可完成提交。


