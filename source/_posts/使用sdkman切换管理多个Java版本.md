---
title: 使用sdkman切换管理多个Java版本
date: 2020-03-12 17:13:08
tags: [Java, 软件]
categories: Java
---

近两年，Java版本更新变得频繁起来，改为每半年发布一个新版本，伴随着每个新版本的发布都会加入新的特性，无论是想要尝鲜还是项目升级，可能要面临管理多个Java版版本的情况。

<!--more-->

在Python里，可以用Anaconda或者miniconda管理多个Python版本。Java的解决方案有三个：

* Jabba
* jenv
* sdkman

这里我用sdkman尝鲜。

官网：[sdkman](<https://sdkman.io/>)

# 安装

```bash
curl -s "https://get.sdkman.io" | bash
```

在此之前如果你没有安装unzip会报错，Ubuntu安装命令：

```bash
apt install zip
apt install unzip
```

之后在终端输入：

```bash
source "$HOME/.sdkman/bin/sdkman-init.sh"
```

最后，查看sdk版本

```bash
sdk version
```

# 使用

## sdk list

查看可用的JDK版本

```bash
sdk list java
```

![](F:\Hexo\source\_posts\assets\Snipaste_2020-03-30_18-40-31.png)

## sdk install

安装一个Java版本，例如我要安装Amazon的JDK版本

```bash
sdk install java 11.0.6-amzn
```

安装完成后，satus就会变成installed状态

## sdk current

```bash
sdk current java
# Using java version 11.0.6-amzn
```

## sdk use

切换到其他Java版本

```bash
sdk use java 14.0.0-zulu
```

## sdk default

指定某个Java版本为默认版本，例如我指定Amazon的Corretto（11.0.6-amzn）为默认版本：

```bash
sdk default java 11.0.6-amzn
# 查看Java版本
Java -version
# openjdk version "11.0.6" 2020-01-14 LTS
# OpenJDK Runtime Environment Corretto-11.0.6.10.1 (build 11.0.6+10-LTS)
# OpenJDK 64-Bit Server VM Corretto-11.0.6.10.1 (build 11.0.6+10-LTS, mixed mode)
```

## sdk uninstall

卸载某个版本

```bash
sdk uninstall java ...
```

## sdk upgrade

```bash
sdk upgrade java
```

# sdkman 卸载

```bash
$ tar zcvf ~/sdkman-backup_$(date +%F-%kh%M).tar.gz -C ~/ .sdkman
$ rm -rf ~/.sdkman
```

之后打开.bashrc，删掉下面

```bash
[[ -s "/home/dudette/.sdkman/bin/sdkman-init.sh" ]] && source "/home/dudette/.sdkman/bin/sdkman-init.sh"
```







