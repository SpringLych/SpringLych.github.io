---
title: 使用Scoop管理Windows软件
date: 2019-05-14 18:42:12
tags: [Windows, 软件]
categories: Windows
---

体验了一下Scoop这款Windows上的软件包管理器，记录下其安装，配置过程。

<!--more-->

## 安装

* [官方网址](https://scoop.sh/)
* [快速入门](<https://github.com/lukesampson/scoop/wiki/Quick-Start>)

我的电脑是Windows 10 1903，满足基础环境。打开PowerShell

1. 保证允许本地脚本运行

   ```powershell
   set-executionpolicy remotesigned -scope currentuser
   ```

2. 如果你不想将scoop将软件默认安装到C盘，可以自定义scoop包安装路径

   ```powershell
    $env:SCOOP='D:\scoop'
    [environment]::setEnvironmentVariable('SCOOP',$env:SCOOP,'User')
   ```

3. 安装scoop

   ```powershell
   iex (new-object net.webclient).downloadstring('https://get.scoop.sh')
   ```

安装完成，使用 `scoop help`验证。出现“Usage: scoop <command> [<args>]

Some useful commands are…”等字样说明安装成功。

之后可以使用 `scoop help`查看命令参考

## 使用

安装软件：`scoop install 软件名`

更新：`scoop update`

移除所有旧版本：`scoop cleanup *`

卸载：`scoop uninstall 软件名`

## 软件仓库

Scoop默认仓库（main bucket）是有限的，但是可以添加仓库。

最常见的bucket - [extras](<https://github.com/lukesampson/scoop-extras>)，其包含各个版本的 Firefox、福昕阅读器、Geek Uninstaller、Inkscape、Snipaste 等等

添加：`scoop bucket add extras `

查找官方维护的仓库：`scoop bucket known`

关于更多Scoop仓库的信息，可以参考少数派[这篇文章](<https://sspai.com/post/52710>)



## 参考

* [少数派 - 「一行代码」搞定软件安装卸载，用 Scoop 管理你的 Windows 软件](<https://sspai.com/post/52496>)
* [Windows | Scoop软件包管理神器](<https://www.limufang.com/post/569.html>)
* [少数派 - 给 Scoop 加上这些软件仓库，让它变成强大的 Windows 软件管理器](<https://sspai.com/post/52710>)

