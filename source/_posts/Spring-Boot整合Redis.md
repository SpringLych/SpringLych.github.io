---
title: Spring Boot整合Redis
date: 2019-05-12 16:01:22
tags: [Spring Boot, Redis]
categories: Spring Boot
---

Spring Boot整合Redis

<!--more-->

## 建立项目

1. 使用IDEA新建Spring Boot项目，依赖选择 web-web，NoSQL-redis。

2. application.properties

   ```properties
   ## Spring Boot启动端口号
   server.port=8080
   
   # Redis服务器地址
   spring.redis.host=149.129.78.187
   # Redis端口
   spring.redis.port=6379
   # Redis密码
   spring.redis.password=
   # Redis数据库索引
   spring.redis.database=0
   # 连接池最大连接数
   spring.redis.jedis.pool.max-active=8
   # 连接池最大阻塞等待时间 负值没有限制
   spring.redis.jedis.pool.max-wait=-1
   # 连接池最大空闲连接
   spring.redis.jedis.pool.max-idle=8
   # 最小空闲连接
   spring.redis.jedis.pool.min-idle=0
   # 连接超时时间
   spring.redis.timeout=100
   ```

3. 测试用例

   ```java
   @RunWith(SpringRunner.class)
   @SpringBootTest
   public class DemoApplicationTests {
   
       @Autowired
       private StringRedisTemplate redisTemplate;
   
       @Test
       public void test() throws Exception{
           redisTemplate.opsForValue().set("a", "aaa");
           Assert.assertEquals("aaa", redisTemplate.opsForValue().get("a"));
       }
   }
   ```

   简单的测试与Redis的连接，使用自动配置的`StringRedisTemplate`对象进行Redis的读写操作


## 参考

* [掘金-SpringBoot整合Redis](<https://juejin.im/post/5ad6acb4f265da239e4e9906>)
* [Spring Boot 使用NoSQL数据库 Redis](<http://www.spring4all.com/article/254>)