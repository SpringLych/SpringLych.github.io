---
title: docker-compose收集
date: 2019-03-26 20:40:58
tags: [Docker,docker-compose]
categories: Docker
---

自己写的，收集整理的一些docker-compose.yml。

<!--more-->

[TOC]



## Aria2

一个多线程下载器，可由网页端管理

```yaml
version: '2'
services:
  aria2:
      image: wahyd4/aria2-ui
      container_name: aria2
      ports: 
        - "6810:80"
        - "6800:6800"
      volumes:
        - /usr/conf/aria2:/root/conf
        - /media/download/aria2:/var/www:rw
          #- /usr/data/aria2:/var/www:rw
      environment:
        - DOMAIN=:80
      restart: always
```

安装完成后通过`http:ip/端口/ui`才能访问到ui页面



## BaiduPCS-web

```yaml
version: '2'
services:
  aria2:
      image: johngong/baidupcs-web
      container_name: baidupcs
      ports: 
        - "5299:5299"
      volumes:
        - /usr/conf/baidupcs:/config
        - /media/download/baidupcs:/root/Downloads
      restart: unless-stopped
```



## filebrowser

[installation](<https://filebrowser.xyz/installation>)

首先需要在你需要准备`config.json`和`database.db`。默认用户名/密码为`admin/admin`

config.json

```json
{
  "port": 80,
  "baseURL": "",
  "address": "",
  "log": "stdout",
  "database": "/database.db",
  "root": "/srv"
}
```

docker-compose.yml

```yaml
version: '2'
services:
  file:
    container_name: filebrowser
    image: filebrowser/filebrowser
    ports:
      - "6880:80"
    volumes:
      - /media:/srv
      - /usr/conf/filebrowser/config.json:/config.json
      - /usr/conf/filebrowser/database.db:/database.db
    restart: unless-stopped
```

## filebrowser-ehanced

```yaml
version: '2'
services:
  fbe:
      image: 80x86/filebrowser
      container_name: fbe
      ports: 
        - "8083:8083"
      volumes:
        - /usr/conf/fbe:/config
        - /srv/dev:/myfiles
      environment:
        - PUID=1000
        - PGID=1000
        - UMASK_SET=000
        - WEB_PORT=8083
      restart: unless-stopped
```

[fbe](<https://hub.docker.com/r/80x86/filebrowser>)



## jellyfin

```yaml
version: "3.7"
services:
  jellyfin:
    image: linuxserver/jellyfin
    container_name: jellyfin
    deploy:
      resources:
        limits:
          cpus: '0.75'
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Shanghai
      - UMASK_SET=022 #optional
    volumes:
      - /config/jellyfin:/config
      - /tvshows:/data/tvshows
      - /movie:/data/movies
      - /transcode:/transcode #optional
    ports:
      - 8096:8096
    restart: unless-stopped

```

启动时加入 `--compatibility` 参数

```bash
docker-compose --compatibility up -d
```





## Kod

[可道云](<https://kodcloud.com/>)

```yaml
version: '2'
services:
  kod:
    image: qinkangdeid/kodexplorer
    container_name: kod
    ports:
      - "80:80"
    volumes:
      - /usr/conf/kod:/var/www/html
      - /media:/data
```

## kafka

 ```yaml
version: '3.0'
services: 
  zookeeper: 
    image: wurstmeister/zookeeper
    ports: 
      - "2181:2181"
  
  kafka:
    image: wurstmeister/kafka
    ports:
      - "9092:9092"
    environment:
      # 注意这里填写宿主的ip
      KAFKA_ADVERTISED_HOST_NAME: 172.18.0.1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_MESSAGE_MAX_BYTES: 20000
      KAFKA_HEAP_OPTS: -Xmx256M -Xms256M
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
 ```

## lychee

基于PHP语言的相册管理程序。这里展示荒野无灯大专为N1优化的镜像,不需要额外的数据库。

```yaml
version: "2"
services:
  lychee:
    image: 80x86/lychee
    container_name: lychee-deng
    environment:
      - PHP_MAX_EXECUTION_TIME=600
      - PHP_TZ=Asia/Shanghai
      - DB_CONNECTION=sqlite
      - DB_DATABASE=/conf/lychee.db
    volumes:
      - /conf/lychee-deng:/conf
      - /download:/uploads
    ports:
      - "80:80"
    restart: unless-stopped

```

[linuxserver版](<https://hub.docker.com/r/linuxserver/lychee>)，需要额外的数据库。

```yaml
version: "2"
services:
  lychee:
    image: linuxserver/lychee
    container_name: lychee-server
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Shanghai
    volumes:
      - /usr/conf/lychee-server:/config
      - /media/download:/pictures
    ports:
      - "80:80"
    restart: unless-stopped

```



## mariadb

```yaml
version: '3.1'
services:
  mariadb:
    image: mariadb
    container_name: mariadb
    restart: always
    environment:
      TZ: Asia/Shanghai
      MYSQL_ROOT_PASSWORD: example
    ports:
      - "3306:3306"
    volumes:
      - /usr/data/mysql:/var/lib/mysql

```







## MongoDB

```yaml
version: '3.1'
services:
  mongo:
    image: mongo
    container_name: mongodb
    restart: always
    environment:
      MONGO_INITDB_ROOT_USERNAME: username
      MONGO_INITDB_ROOT_PASSWORD: password
    ports:
      - "27017:27017"
    volumes:
      - /usr/data/mongodb:/data/db

```

MongoDB删除重复字段

```sql
db.climage.aggregate([
    {
        $group: {
            _id: {
                title: '$title',
                page_url: '$page_url'
            },
            count: {
                $sum: 1
            },
            dups: {
                $addToSet: '$_id'
            }
        }
    },
    {
        $match: {
            count: {
                $gt: 1
            }
        }
    }
]).forEach(function(doc) {
    doc.dups.shift();
    db.climage.remove({
        _id: {
            $in: doc.dups
        }
    });
});
```

## NextCloud

```yaml
version: "2"
services:
  nextcloud:
    image: linuxserver/nextcloud
    container_name: nextcloud
    environment:
      - PUID=0
      - PGID=0
      - TZ=Asia/Shanghai
    volumes:
      - /usr/conf/nextcloud:/config
      - /media:/data
    ports:
      - "80:80"
    restart: unless-stopped
```



## qBittorrent

```yaml
version: "2"
services:
  qbittorrent:
    image: linuxserver/qbittorrent
    container_name: qbittorrent
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Shanghai
      - UMASK_SET=022
      - WEBUI_PORT=8080
    volumes:
      - /usr/conf/qbit:/config
      - /downloads:/downloads
    ports:
      - 6881:6881
      - 6881:6881/udp
      - 8080:8080
    restart: unless-stopped

```

[linuxserver/qbittorrent ](<https://hub.docker.com/r/linuxserver/qbittorrent>)

其中，8080端口为webUI端口。

在宿主机和容器中会发生用户不一致导致的权限问题，因此定义PUID和PGID来解决权限问题。

例如，在宿主机有一个用户dockeruser和用户组dockergroup，其uid=100，gid=100，那么指定PUID=100和PGID=100，则容器中的相关目录拥有和dockeruser，dockergroup同样的权限。

该镜像支持x86-64,arm64和armhf，拉取是会自动拉取合适的架构镜像，也可指定tag拉取。

注意volumes那里要对应downloads，不然PUID和PGID不会生效。

升级：

```shell
docker-compose up -d qbittorrent
```





## RabbitMq

```yaml
version: '3.0'
services:
  rabbitmq:
      image: rabbitmq:management-alpine
      container_name: rabbitmq
      environment:
        - RABBITMQ_DEFAULT_USER=root
        - RABBITMQ_DEFAULT_PASS=654321
      ports:
        - "1560:15672"
        - "5672:5672"
      volumes:
        - ./data:/var/lib/rabbitmq
      restart: always
```

5672：rabbitmq服务端口

1560：rabbitmq网页端口

## Redis

```yaml
version: '2'
services:
  redis:
    image: redis
    container_name: redis
    ports:
        - 6379:6379
    volumes:
      - /usr/conf/redis/redis.conf:/usr/local/etc/redis/redis.conf
      - /usr/conf/redis/data:/data  
    restart: unless-stopped
```

## Webdav

```yaml
version: '3'
services:
  webdav:
    image: bytemark/webdav
    container_name: webdav
    restart: always
    ports:
      - "80:80"
    environment:
      AUTH_TYPE: Digest
      USERNAME: username
      PASSWORD: pass
    volumes:
      - /media/download:/var/lib/dav/data
```









