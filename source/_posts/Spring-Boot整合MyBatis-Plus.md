---
title: Spring Boot整合MyBatis-Plus
date: 2019-05-24 14:46:13
tags: [Spring Boot, MyBatis]
categories: Spring Boot
---

MyBatis-Plus是MyBatis的增强工具。

<!--more-->

## 介绍

官网：[官网](<https://mybatis.plus/>)

由于Mybatis plus完全基于Mybatis,且吸收了一部分HIbernate的优点,提供集成的CRUD,基本上现在开发中都使用Mybatis Plus替代原生Mybatis框架

**MybatisPlus 不能和 Mybatis同时使用**

## 使用

### 添加依赖

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-jdbc</artifactId>
    </dependency>
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <scope>runtime</scope>
    </dependency>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>

    <dependency>
        <groupId>com.baomidou</groupId>
        <artifactId>mybatis-plus-boot-starter</artifactId>
        <version>3.1.1</version>
    </dependency>
</dependencies>
```

### 数据

数据库 Schema 脚本:

```mysql
DROP TABLE IF EXISTS user;

CREATE TABLE user
(
	id BIGINT(20) NOT NULL COMMENT '主键ID',
	name VARCHAR(30) NULL DEFAULT NULL COMMENT '姓名',
	age INT(11) NULL DEFAULT NULL COMMENT '年龄',
	email VARCHAR(50) NULL DEFAULT NULL COMMENT '邮箱',
	PRIMARY KEY (id)
);
```

其对应的数据库 Data 脚本如下：

```mysql
DELETE FROM user;

INSERT INTO user (id, name, age, email) VALUES
(1, 'Jone', 18, 'test1@baomidou.com'),
(2, 'Jack', 20, 'test2@baomidou.com'),
(3, 'Tom', 28, 'test3@baomidou.com'),
(4, 'Sandy', 21, 'test4@baomidou.com'),
(5, 'Billie', 24, 'test5@baomidou.com');
```

### 配置文件

可以参考[Spring-Boot整合MyBatis]()

```yml
server:
  port: 8080
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/springboot_mybatis?serverTimezone=UTC&useUnicode=true&characterEncoding=utf-8&useSSL=false
    username: root
    password: 654321
    driver-class-name: com.mysql.cj.jdbc.Driver
```

* 注：这里`driver-class-name`使用的是`com.mysql.cj.jdbc.Driver`，`com.mysql.cj.jdbc.Driver` 是 mysql-connector-java 6中的，需要指定时区serverTimezone。

### 业务

设置。指定mapper地址，启用分页功能。

```java
@Configuration
@MapperScan("com.example.demo.mapper")
public class MyBatisPlusConfig {
    @Bean
    public PaginationInterceptor paginationInterceptor() {
        return new PaginationInterceptor();
    }
}
```

使用lombok创建实体类。

```java
@Data
public class User {
    private Long id;
    private String name;
    private Integer age;
    private String email;
}
```

创建mapper

```java
public interface UserMapper extends BaseMapper<User> {
}
```

### 测试

```java
@SpringBootTest
@RunWith(SpringRunner.class)
public class UserMapperTest {
    private Logger logger = LoggerFactory.getLogger(this.getClass());

    @Autowired
    private UserMapper userMapper;
}
```

查询一个和查询全部

```java
@Test
public void testSelectOne() {
    User user = userMapper.selectById(1L);
    logger.info("user = {}", user);
}

@Test
public void testSelectAll() {
    List<User> list = userMapper.selectList(null);
    list.forEach(user -> {
        logger.info("user = {}", user);
    });
}
```

分页查询

```java
@Test
public void testPage(){
    logger.info("----------分页---------");
    Page<User> page = new Page<>(1,2);
    IPage<User> userIPage = userMapper.selectPage(page,
                                                  new QueryWrapper<User>().gt("age", 6));
    logger.info("总条数：{}", userIPage.getTotal());
    logger.info("当前页数：{}", userIPage.getCurrent());
    logger.info("当前每页显示数：{}", userIPage.getSize());
    logger.info("----------分页---------");
}
```

更新

```java
@Test
public void update() {
    User user = userMapper.selectById(2L);
    assertThat(user.getAge()).isEqualTo(20);

    userMapper.update(
        null,
        Wrappers.<User>lambdaUpdate().set(User::getEmail, "123@123").eq(User::getId, 2)
    );
    assertThat(userMapper.selectById(2).getEmail()).isEqualTo("123@123");
}
```

更多CRUD接口可以查看这里：[CRUD接口](<https://mybatis.plus/guide/crud-interface.html#mapper-crud-%E6%8E%A5%E5%8F%A3>)

## 参考

* [官网](<https://mybatis.plus/guide/#%E7%89%B9%E6%80%A7>)
* [纯洁的微笑 - Spring Boot 2 (十一)：如何优雅的使用 MyBatis 之 MyBatis-Plus](<http://www.ityouknow.com/springboot/2019/05/14/spring-boot-mybatis-plus.html>)