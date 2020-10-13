---
title: Lombok安装及使用
date: 2019-05-23 14:20:12
tags: [Java]
categories: [Java]
---

使用IDEA安装Lombok及其使用。

<!--more-->

## Lombok是什么

Lombok是一个通过注解以达到减少代码的Java库,如通过注解的方式减少get,set方法,构造方法等。

官网：[Project Lombok](<https://projectlombok.org/>)

## 安装

首先向`pom.xml`中添加依赖：

```xml
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.8</version>
    <scope>provided</scope>
</dependency>
```

其中 `scope=provided`，就说明 `Lombok` 只在编译阶段生效。也就是说，`Lombok` 会在编译期静悄悄地将带 `Lombok` 注解的源码文件正确编译为完整的 `class` 文件。


### IDEA插件

通过安装插件，让IDEA能够识别。安装过程可参考官网：[setup - IntelliJ IDEA](<https://projectlombok.org/setup/intellij>)

## 使用

常用注解

* @var
* @Data

* @Setter @Getter
* @NonNull
* @Synchronized
* @ToString
* @Log
* @EqualsAndHashCode
* @Cleanup
* @SneakyThrows

### @var

> Mutably! Hassle-free local variables.

```java
public void test() {
    var list = new ArrayList<String>(10);
    list.add("Hello");
    list.add("Ming");
    var one = list.get(0);
    System.out.println(one);

    for (var a : list) {
        System.out.println(a);
    }
}
```

```java
public void test() {
    ArrayList<String> list = new ArrayList<>(10);
    list.add("Hello");
    list.add("Ming");
    String one = list.get(0);
    System.out.println(one);

    for (String a : list) {
        System.out.println(a);
    }
}
```

### @Data

>  All together now: A shortcut for `@ToString`, `@EqualsAndHashCode`, `@Getter` on all fields, and `@Setter` on all non-final fields, and `@RequiredArgsConstructor`!

作用于类。相当于同时加上以下注解@Setter @Getter,@ToString,@EqualsAndHashCode

```java
@Data
public class User {
    private Long id;
    private String name;
    private String sex;
    private String address;
}
```

### @Getter/@Setter

> Never write `public int getFoo() {return foo;}` again.

作用于属性。

```java
public class User {
    @Getter@Setter
    private String name;
}
```

相当于

```java
public class User {
    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```

### @NonNull

> You can use `@NonNull` on the parameter of a method or constructor to have lombok generate a null-check statement for you.

该注解快速判断是否为空,如果为空,则抛出java.lang.NullPointerException

官网例子：

```java
public class NonNullExample extends Something {
  private String name;
  
  public NonNullExample(@NonNull Person person) {
    super("Hello");
    this.name = person.getName();
  }
}
```

```java
public class NonNullExample extends Something {
  private String name;
  
  public NonNullExample(@NonNull Person person) {
    super("Hello");
    if (person == null) {
      throw new NullPointerException("person is marked @NonNull but is null");
    }
    this.name = person.getName();
  }
}
```

### @Synchronized

作用范围为方法。自动添加到同步机制,有趣的是,生成的代码锁代码块

```java
private DateFormat format = new SimpleDateFormat("MM-dd-YYYY");

@Synchronized
public String synchronizedFormat(Date date) {
    return format.format(date);
}
```

```java
private final java.lang.Object $lock = new java.lang.Object[0];
private DateFormat format = new SimpleDateFormat("MM-dd-YYYY");

public String synchronizedFormat(Date date) {
    synchronized ($lock) {
        return format.format(date);
    }
}
```

### @ToString

可以进一步设置：

* callSuper 是否输出父类的toString方法,默认为false
* includeFieldNames 是否包含字段名称,默认为true
* exclude 排除生成tostring的字段

```java
@ToString(callSuper = true, exclude = {"address"})
public class User {
    private String name;
    private String address;
}
```

### @Log

可以使用Log4j，Log4j2，Slf4j等。具体看[官方文档](<https://projectlombok.org/features/log>)

**SpringBoot选用 SLF4j和logback**

```java
@Slf4j
public class LogEx{
    public void test(){
        log.error("Error")
    }
}
```

### @Cleanup

用于确保已分配的资源被释放,如IO的连接关闭。

```java
public void testCleanUp() {
    try {
        @Cleanup ByteArrayOutputStream baos = new ByteArrayOutputStream();
        baos.write(new byte[] {'Y','e','s'});
        System.out.println(baos.toString());
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```



## 参考

* 官方文档：[Lombok features](<https://projectlombok.org/features/all>)
* [Lombok使用详解](<https://www.andyqian.com/2017/04/22/java/tools/Lombok%E8%AF%A6%E8%A7%A3/>)

