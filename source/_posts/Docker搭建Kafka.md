---
title: Docker搭建Kafka
date: 2019-07-26 22:41:30
tags: [Docker, docker-compose, Kafka]
categories: Docker
---

准备在项目中学习Kafka，因此准备在Linux环境下使用Docker搭建Kafka环境。

<!--more-->

## 准备

我使用的是Ubuntu 16.04，已经安装好Docker和Docker Compose。Kafka没有官方镜像，但是有知名的第三方镜像[wurstmeister/kafka ](<https://hub.docker.com/r/wurstmeister/kafka>)，同时，需要用zookeeper管理kafka，因此可以用[wurstmeister/zookeeper ](<https://hub.docker.com/r/wurstmeister/zookeeper>)

## 使用docker-compose

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
      KAFKA_HEAP_OPTS: -Xmx256M -Xms256M
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
```

* 指定`KAFKA_HEAP_OPTS`是由于我的主机内存小，需要限制内存，否则无法启动。如果主机内存足够，不需要限制

* `KAFKA_ADVERTISED_HOST_NAME`需要用到宿主机的IP。在Ubuntu18.04中，每次登陆都会显示系统信息，docker0就是填写的宿主机ip。

![](1.png)

	在Ubuntu16.04中，可以使用下面的命令查询：

```shell
ip addr show docker0
```

查询结果为：172.18.0.1

启动kafka

```shell
docker-compose up -d
```

之后通过 `docker ps `可以看到启动了一个zookeeper容器和一个kafka容器。

## 验证

1. 进入到kafka容器中

```shell
docker exec -it kafka_kafka_1 /bin/bash
```

2. 创建一个topic，

```shell
bash-4.4> kafka-topics.sh --create --topic test --zookeeper kafka_zookeeper_1:2181 --replication-factor 1 --partitions 1
# Created topic test.
bash-4.4> kafka-topics.sh --list --zookeeper kafka_zookeeper_1:2181
# test
```

3. 发布几条消息

```shell
bash-4.4> kafka-console-producer.sh --topic=test --broker-list kafka_kafka_1:9092
>hello this is kafka
>wow awesome
>hahah
>hello r
^c退出
```

4. 接受消息

```shell
bash-4.4> kafka-console-consumer.sh --bootstrap-server kafka_kafka_1:9092 --from-beginning --topic test
hello this is kafka
wow awesome
hahah
hello r
```

![](2.png)

接收到了发布的消息，部署正常。

## 待解决

使用Spring Boot整合Kafka时发现无法连接，一直报错`[Producer clientId=producer-1] Connection to node 1001 could not be established. Broker may not be available.`显示Google了好久暂时未解决，猜测是由于使用Docker，内外网ip没有设置正确，导致外网无法访问。

## 参考

* [使用Docker快速搭建Kafka开发环境](<https://tomoyadeng.github.io/blog/2018/06/02/kafka-cluster-in-docker/index.html>)
* [StackOverflow - kafka 8 and memory - There is insufficient memory for the Java Runtime Environment to continue](https://stackoverflow.com/questions/21448907/kafka-8-and-memory-there-is-insufficient-memory-for-the-java-runtime-environme)

