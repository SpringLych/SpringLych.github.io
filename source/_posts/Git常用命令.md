---
title: Git常用命令
date: 2019-03-28 14:10:19
tags: Git
categories: Git
---

整理一些常用的git命令

<!--more-->

## 本地push到远程

初始化本地

```bash
git init
```

本地关联远程

```bash
git remote add origin git@gitres
```

添加和提交

```bash
git add .
git commit -m "注释"
```

push 远程

```bash
第一次推送master分支时，加上-u参数，Git不但会把本地的master分支内容推送的远程新的master分支，还会把本地的master分支和远程的master分支关联起来，
git push -u  origin master

git push origin master
```

## 文件操作

### 添加文件目录

```bash
# 添加当前目录所有文件到暂存区
git add .
git add [file1] [file2]...
git add [dir]
```

### 撤销add

```bash
git rm --cached <file>
```

