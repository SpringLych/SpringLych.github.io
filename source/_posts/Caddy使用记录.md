---
title: Caddy使用记录
date: 2019-06-23 22:59:05
tags: [Caddy]
categories: Caddy
---

Caddy是使用Go语言编写的一个轻量的web服务器，类似Nginx。

<!--more-->

[官网](<https://caddyserver.com/>)

Caddy轻量，配置简单，插件丰富，我一般使用用来折腾自己的VPS，这里记录下其安装过程和配置文件

## 安装

使用官方提供的脚本安装

```bash
curl https://getcaddy.com | bash -s personal
```

带插件

```bash
curl https://getcaddy.com | bash -s personal http.webdav
```

## 配置

Caddyfile

```
:6600 {
  root /data
  timeouts none
  gzip
}
```

## 运行

```bash
caddy -conf /path/Caddyfile
```

也可以使用nohup后台运行

```bash
nohup caddy -conf /path/Caddyfile &
```



## 插件

### webdav

```
:6600 {
  root /media
  timeouts none
  gzip
  basicauth / user password
  webdav {
    scope /media #需要显示的路径
  }
}

```

