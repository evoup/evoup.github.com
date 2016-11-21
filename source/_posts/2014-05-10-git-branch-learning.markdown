---
layout: post
title: "git分支学习"
date: 2014-05-10 14:17
comments: true
categories: [version_control]
---


主要学习的内容：git默认分支是什么，如何创建分支，如何查看所有分支，如何查看当前分支，如何提交分支，如何切换到分支,以下简单给出答案：
<!-- more -->

git的默认分支是master

创建分支

```bash
git branch branchname 
```

显示所有分支

```bash
git show-branch
```

查看当前分支

```bash
git status
```

提交分支

```bash
git push origin branchname
```

切换到名字为branchname的分支

```bash
git checkout branchname
```


创建一个分支，然后把这个分支同步到远程仓库

```bash
git branch branchname
git checkout branchname
git push origin branchname
```

在master分支创建一个分支，直接切换到该分支(假设已经在在master分支)

```bash
git checkout -b branchname
```

修改分支名

```bash
git branch -m oldbranchname newbranchname
```

删除分支

```bash
git branch -d branchname
```

推送分支

```bash
git push origin branchname
```


