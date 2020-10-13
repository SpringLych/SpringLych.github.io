---
title: IDEA构建第一个Spring Boot项目
date: 2019-05-12 15:34:51
tags: [Spring Boot]
categories: Spring Boot
---

使用IDEA快速构建一个Spring Boot项目，并将其打包成jar文件。

<!--more-->

## HelloWorld实战

1. 新建项目。IDEA中选择Spring Initializ，填写Group等信息。选择依赖，选择web-web，finnish完成项目创建。

2. 项目结构说明：

   * xxxApplication：main()方法，用于启动
   * xxxApplicationTests：空的Junit测试
   * application.properties：空文件，用于配置属性
   * pom.xml：Maven构建说明文件

3. Controller。新建controller包，并新建HelloWorldController.java文件，代码如下：

   ```java
   @RestController
   public class HelloWorldController {
       
       @RequestMapping("/")
       public String hello(){
           return "Hello World";
       }
   }
   ```

   @RestController和@RequestMapping注解是来自SpringMVC的注解，它们不是SpringBoot的特定部分。

   1. **@RestController**：提供实现REST API可以服务JSON,XML或者其他。这里是以String的形式渲染出结果。它是 @Controller 和 @ResponseBody 注解的合体版
   2. **@RequestMapping**：提供路由信息，”/“路径的HTTP Request都会被映射到hello方法进行处理。

4. 启动应用。浏览器访问<http://localhost:8080/>，浏览器输出 “Hello World”

5. 测试类。

   ```java
   public class HelloWorldControllerTest {
   
       @Test
       public void hello() {
           assertEquals("Hello World", new HelloWorldController().hello());
       }
   }
   ```

## 打包

1. 在`pom.xml`中添加`<packaging>jar</packaging>`，指定打成的包为jar。

2. 整个pom.xml文件为

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
       <modelVersion>4.0.0</modelVersion>
       <parent>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-starter-parent</artifactId>
           <version>2.1.4.RELEASE</version>
           <relativePath/> <!-- lookup parent from repository -->
       </parent>
       <groupId>com.demo</groupId>
       <artifactId>demo</artifactId>
       <version>0.0.1-SNAPSHOT</version>
       <packaging>jar</packaging>
       <name>demo</name>
       <description>Demo project for Spring Boot</description>
   
       <properties>
           <java.version>1.8</java.version>
       </properties>
   
       <dependencies>
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-web</artifactId>
           </dependency>
   
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-test</artifactId>
               <scope>test</scope>
           </dependency>
       </dependencies>
   
       <build>
           <plugins>
               <plugin>
                   <groupId>org.springframework.boot</groupId>
                   <artifactId>spring-boot-maven-plugin</artifactId>
               </plugin>
           </plugins>
       </build>
   
   </project>
   ```

3. 与 pom.xml文件同级运行 `mvn clean package`，首先要确保Maven正确安装。打包成功后，在项目目录下target文件夹中生成jar

4. 进入target文件夹，使用`java -jar jar包名称`即可运行Spring Boot项目

## 参考

* [Spring Boot 之 HelloWorld详解](https://www.bysocket.com/archives/1124/spring-boot-%e4%b9%8b-helloworld%e8%af%a6%e8%a7%a3)
* [SpringBoot简单打包部署(附工程)](<https://juejin.im/post/5b71a72e51882561446fb1d2>)