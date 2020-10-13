---
title: Spring Boot整合Kafka
date: 2019-07-27 15:17:55
tags: [Spring Boot, Kafka]
categories: Spring Boot
---

整合Kafka。

<!--more-->

## Kafka介绍

Kafka是一种高吞吐量的分布式发布订阅消息系统，是消息中间件的一种。它一些基本术语：

1. Kafka将消息以**topic**为单位进行归纳
2. 向Kafka topic发布消息的程序称为**producers**
3. 预定topic并消费消息的程序称为**consumer**
4. Kafka以集群方式运行，可由一个或多个服务组成，每个服务叫做一个broker

前面已经搭建好了Kafka，[搭建过程](<https://springlych.github.io/2019/07/26/Docker%E6%90%AD%E5%BB%BAKafka/>)

## Spring Boot整合Kafka

使用IDEA构建项目Spring Boot项目，选择依赖message-Apache Kafka。

### 实体类

```java
@Data
public class Message implements Serializable {
    private String form;
    private String message;

    public Message() {
    }

    public Message(String form, String message) {
        this.form = form;
        this.message = message;
    }
}
```

### 配置

`application.yml`。Spring Boot支持在application.yml中配置Kafka的生产者配置和消费者配置。当然也可以使用`@Configuration`使用Java类配置。

```yaml
spring:
  kafka:
    consumer:
      bootstrap-servers: localhost:9092
      group-id: test-consumer
      auto-offset-reset: latest
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      value-deserializer: org.apache.kafka.common.serialization.StringDeserializer
    producer:
      bootstrap-servers: localhost:9092
      key-serializer: org.apache.kafka.common.serialization.StringDeserializer
      value-serializer: org.apache.kafka.common.serialization.StringDeserializer
```

### 创建生产者

发送消息

```java
@Service
public class Producer {
    private Logger logger = LoggerFactory.getLogger(this.getClass());
    private static final String TOPIC = "users";

    @Autowired
    private KafkaTemplate<String, String> kafkaTemplate;

    public void sendMessage(String message) {
        ListenableFuture<SendResult<String, String>> future
                = this.kafkaTemplate.send(TOPIC, message);
        future.addCallback(new ListenableFutureCallback<SendResult<String, String>>() {
            @Override
            public void onFailure(Throwable ex) {
                logger.error("消息：{} 发送失败，原因：{}", message, ex.getMessage());
            }

            @Override
            public void onSuccess(SendResult<String, String> result) {
                logger.info("成功发送消息：{}, offset=[{}]", message, result.getRecordMetadata());
            }
        });
    }
}
```

在send方法中，通过回调的方式确定消息是否发送成功。

我们没有配置KafkaTemplate，因此使用`@Service`自动配置。

### 创建消费者

接受消息

```java
public class Consumer {
    private Logger logger = LoggerFactory.getLogger(this.getClass());

    @KafkaListener(topics = "users")
    public void consumer(String message) throws IOException{
        logger.info("接受消息：{}", message);
    }
}
```

`@KafkaListener`注解来监听名称为users的Topic

`bootstrap-servers`:生产者消费者的地址。

`key-deserializer`和`value-deserializer`指定了序列化策略，生产者指定为kafka提供的StringSerializer，用来发送简单的String类消息

### 发布消息

```java
@Service
@RestController
public class SendMessageController {
    private final Producer producer;

    @Autowired
    SendMessageController(Producer producer) {
        this.producer = producer;
    }

    @GetMapping("send/{message}")
    public void send(@PathVariable String message) {
        this.producer.sendMessage(message);
    }
}
```

### 错误

启动后，访问`http://localhost:8080/send/hello`，发现报告一下错误：

> 消息：hello,mrbird 发送失败，原因：Failed to send; nested exception is org.apache.kafka.common.errors.TimeoutException: Expiring 1 record(s) for users-0: 30047 ms has passed since batch creation plus linger time
>
> [Producer clientId=producer-1] Connection to node 1001 could not be established. Broker may not be available.

猜测是由于使用Docker创建了Kafka，导致外网访问Kafka时没有配置好正确的IP。该问题待解决。

## 参考

* [How to Work with Apache Kafka in Your Spring Boot Application](<https://www.confluent.io/blog/apache-kafka-spring-boot-application>)
* [MrBird - Spring Boot整合Kafka](<https://mrbird.cc/Spring-Boot-Kafka.html>)