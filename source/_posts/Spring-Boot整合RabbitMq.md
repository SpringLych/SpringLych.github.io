---
title: Spring Boot整合RabbitMq
date: 2019-08-06 20:32:27
tags: [Spring Boot]
categories: [Spring Boot]
---

整合RabbitMQ

<!--more-->

启动IDEA，初始化Spring Boot，选择message - rabbitmq

## 配置文件

```yaml
spring:
  rabbitmq:
    host: rabbitmq ip地址
    port: 5672
    username: root
    password: pass

```

## Rabbit设置

RabbitMqConfig

```java
@Configuration
public class RabbitMqConfig {

    /**
     * 一对一
     */
    @Bean
    public Queue queueOne() {
        return new Queue(RabbitConst.QUEUE_ONE);
    }

    /**
     * 用来测试一对多
     */
    @Bean
    public Queue queueMany() {
        return new Queue(RabbitConst.QUEUE_MANY);
    }

    /**
     * 对象的发送和接受
     */
    @Bean
    public Queue objectQueue() {
        return new Queue(RabbitConst.QUEUE_OBJECT);
    }
}
```

## 一对一测试

发送者

rabbitTemplate 是 Spring Boot 提供的默认实现

```java
@Component
public class HelloSender {
    private final AmqpTemplate rabbitTemplate;
    private Logger logger = LoggerFactory.getLogger(this.getClass());

    public HelloSender(AmqpTemplate rabbitTemplate) {
        this.rabbitTemplate = rabbitTemplate;
    }

    public void send(){
        String context = "send hello" + new Date();
        logger.info("Send: {}", context);
        this.rabbitTemplate.convertAndSend(RabbitConst.QUEUE_ONE, context);
    }
}

```

接受者

通过@RabbitListener注解要接收消息的方法

```java
@Component
@RabbitListener(queues = RabbitConst.QUEUE_ONE)
public class HelloReceiver {
    private Logger logger = LoggerFactory.getLogger(this.getClass());

    @RabbitHandler
    public void process(String hello) {
        logger.info("HelloReceive {}", hello);
    }
}
```

## 高级使用

### 对象支持

```java
/**
发送者
*/
@Component
public class ObjectSender {
    private Logger logger = LoggerFactory.getLogger(this.getClass());
    private final AmqpTemplate rabbitTemplate;

    public ObjectSender(AmqpTemplate rabbitTemplate) {
        this.rabbitTemplate = rabbitTemplate;
    }

    public void send(User user) {
        logger.info("-----ObjectSender send user: {}-----", user.toString());
        rabbitTemplate.convertAndSend(RabbitConst.QUEUE_OBJECT, user);
    }
}

/**
接受者
*/
@Component
@RabbitListener(queues = RabbitConst.QUEUE_OBJECT)
public class ObjectReceiver {
    private Logger logger = LoggerFactory.getLogger(this.getClass());

    @RabbitHandler
    public void process(User user) {
        logger.info("-----receive user: {} -----", user.toString());
    }
}

```

![1565096910459](1565096910459.png)



## 参考

* [纯洁的微笑 - Spring Boot(八)：RabbitMQ 详解](<http://www.ityouknow.com/springboot/2016/11/30/spring-boot-rabbitMQ.html>)