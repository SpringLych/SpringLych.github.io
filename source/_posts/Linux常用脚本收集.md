---
title: Linux常用脚本收集
date: 2019-05-30 09:58:49
tags: [Linux]
categories: Linux
---

包括Docker, Docker-compose

<!--more-->

## 服务器评测

```shell
wget -qO- git.io/superbench.sh | bash
```



## 安装Docker

适用于Ubuntu 18.04

```shell
apt-get remove docker docker-engine docker.io containerd runc
apt-get update
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable"
sudo apt update
apt-get install docker-ce docker-ce-cli containerd.io
docker --version
```

如果出现 `unable to resolve host ubuntu`，首先查看`/etc/hostname`中的内容，之后编辑`/etc/hosts`，添加以下内容：`127.0.1.1		ubutnu`

## 安装docker-compose

```shell
sudo curl -L "https://github.com/docker/compose/releases/download/1.23.2/docker-compose-Linux-x86_64" -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
docker-compose --version
echo 'docker docker-compose installed'
```

## 安装 Miniconda

```shell
# 下载
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
```

之后执行脚本，选择安装位置（我一般选`/usr/local/bin/miniconda3`）

## plus

```shell
wget -N --no-check-certificate "https://raw.githubusercontent.com/chiakge/Linux-NetSpeed/master/tcp.sh" && chmod +x tcp.sh && ./tcp.sh
```

## Rclone

使用Rclone挂载Onedriver网盘

[官网安装教程](<https://rclone.org/install/>)

```shell
curl https://rclone.org/install.sh | sudo bash
```

客户端授权。

需要在本地Windows上下载rclone。[下载](https://rclone.org/downloads/)。解压。使用cmd进入rclone文件夹。执行`reclone authorize “onedrive”`，选择，登录onedriver账号后会出现`{'accss_token':'xxx'}`，复制xxx内容



之后在Linux上通过脚本安装。安装完成执行：

> rclone config

```shell
No remotes found - make a new one
n) New remote
s) Set configuration password
q) Quit config
n/s/q> n
name> spring
Type of storage to configure.
Enter a string value. Press Enter for the default ("").
Choose a number from below, or type in your own value
 ....
19 / Microsoft OneDrive
   \ "onedrive"

Storage> 19
** See help for onedrive backend at: https://rclone.org/onedrive/ **

Microsoft App Client Id
Leave blank normally.
Enter a string value. Press Enter for the default ("").
client_id> 
Microsoft App Client Secret
Leave blank normally.
Enter a string value. Press Enter for the default ("").
client_secret> 
Edit advanced config? (y/n)
y) Yes
n) No
y/n> n
Remote config
Use auto config?
 * Say Y if not sure
 * Say N if you are working on a remote or headless machine
y) Yes
n) No
y/n> n
For this to work, you will need rclone available on a machine that has a web browser available.
Execute the following on your machine:
	rclone authorize "onedrive"
Then paste the result below:
result> ## 刚才复制的token
Choose a number from below, or type in an existing value
 1 / OneDrive Personal or Business
   \ "onedrive"
Your choice> 1
Found 1 drives, please select the one you want to use:
0:  (personal) id=xxx
Chose drive to use:> 0
Found drive 'root' of type 'personal', URL: https://onedrive.live.com/?cid=xx
Is that okay?
y) Yes
n) No
y/n> y
--------------------
[spring]
type = onedrive
token = xxx
drive_id = xxx
drive_type = personal
--------------------
y) Yes this is OK
e) Edit this remote
d) Delete this remote
y/e/d> y
Current remotes:

Name                 Type
====                 ====
xxx               onedrive

e) Edit existing remote
n) New remote
d) Delete remote
r) Rename remote
c) Copy remote
s) Set configuration password
q) Quit config
e/n/d/r/c/s/q> q

```

挂载

> rclone mount DriveName:Folder LocalFolder --copy-links --no-gzip-encoding --no-check-certificate --allow-other --allow-non-empty --umask 00

这样挂载后会卡住不动，因此可以让其在后台运行：

> nohup rclone mount DriveName:Folder LocalFolder --copy-links --no-gzip-encoding --no-check-certificate --allow-other --allow-non-empty --umask 00 &

之后使用`df -h`可以查看