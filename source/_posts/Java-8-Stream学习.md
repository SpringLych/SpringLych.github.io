---
title: Java 8 Stream学习
date: 2020-03-30 16:00:25
tags: Java
categories: Java
---

Stream学习。

<!--more-->

# 什么是Stream

Stream和lambda都是Java 8中重要的新特性。它允许你以**声明式**的方式处理数据集合

使用Stream有3步：

1. 创建Stream流
2. 通过Stream流对象执行中间操作
3. 执行最终操作，得到结果

Stream中的源包括集合，数组，文件，正则表达式匹配器，随机数和其他流。流中的数据元素可以是对象引用或基本类型包括**int，long，double**。

# 第一步：Stream创建

创建空的Stream

```java
Stream<String> stream = Stream.empty();
IntStream intStream = IntStream.empty();
```

## IntStreams

有以下几种方法：

* 使用`IntStream.of()`方法
* 使用`IntStream.range(a,b)`（不包含b）和`IntStream.rangeClosed(a,b)`(包含b)
* `IntStream.iterate(1, i -> i + 2)`

```java
public static void main(String[] args) {
    // 1.
    IntStream intStream = IntStream.of(1, 2, 3);
    // 2.
    IntStream intStream1 = IntStream.range(1, 5);
    IntStream intStream2 = IntStream.rangeClosed(1, 5);
    // 3. 从1 开始，后面每个元素依次+2
    IntStream intStream3 = IntStream.iterate(1, i -> i + 2);
    // 4. 每个字符的int值
    IntStream chars = "ABCD".chars();
    // 5. 生成5个1...10范围内的随机数
    IntStream intStream4 = new Random().ints(5, 1, 10);
}
```

## 数组流

基于现有的数组进行创建，或者基于of()方法

```java
String[] arr = {"Alex", "Jon", "Chandler", "Lion"};
Stream<String> stream = Stream.of(arr);
Stream<String> stream1 = Stream.of("Alex", "Jon", "Chandler", "Lion");
```

## 集合流

```java
List<String> list = Arrays.asList("Alex", "Jon", "Chandler", "Lion");
Stream<String> stream = list.stream();
Set<String> set = new HashSet<>(list);
Stream<String> streamSet = set.stream();
```

## 从文件读取

```java
Stream<String> lines = Files.lines(Paths.get("file.txt"));
```

# 第二步：Stream处理

## Filter

将你要处理的元素集合缩小一定的范围，从而为后续的操作减轻压力，例如下面的代码是针对首字母进行过滤

```java
String[] names = {"Alex", "Lion", "Linda", "Chandler", "Luck"};
Stream<String> stream = Stream.of(names).filter(s -> s.startsWith("L"));
```

## Skip

选择跳过前面N个元素来直接访问后面的元素

```java
Stream<String> stream1 = Stream.of(names).skip(2);
stream1.forEach(System.out::println);
// Linda Chandler Luck
```

## Distinct

去重

## Sorted

使用Java默认排序方式

```java
String[] names = {"Alex", "Lion", "Linda", "Chandler", "Luck"};
Stream<String> stream = Stream.of(names).sorted();
stream.forEach(e -> System.out.print(e + " "));
// Alex Chandler Linda Lion Luck 
```

 comparator 进行排序

```java
Stream<String> stream = Stream.of(names).sorted(Comparator.comparing(String::length));
stream.forEach(e -> System.out.print(e + " "));
// Alex Lion Luck Linda Chandler
```

## Map

将Stream 的元素映射到另一个值或类型，它可以将其转换为其他元素。这意味着此操作的结果可以是任何类型的Stream

```java
Stream<String> stream = Stream.of(names).map(String::toUpperCase);
stream.forEach(e -> System.out.print(e + " "));
// ALEX LION LINDA CHANDLER LUCK
```

转换类型

* .mapToInt();
* .mapToDouble();
* .mapToLong();

```java
IntStream stream = Stream.of(names).mapToInt(String::length);
stream.forEach(e -> System.out.print(e + " "));
// 4 4 5 8 4 
```

# 第三步：执行最终操作

最终操作就是产生结果结束Stream流程。类似forEach()或count()。

```java
String[] names = {"Alex", "Lion", "Linda", "Chandler", "Luck"};
Stream<String> stream = Stream.of(names).skip(2);
stream.forEach(System.out::println);
```

## Collection

通过`collect()`将Stream处理好的结果放置到某个地方，如Set。

```java
String[] names = {"Alex", "Lion", "Linda", "Chandler", "Luck"};
// To Set
Set<String> collectSet = Stream.of(names).collect(Collectors.toSet());
// To List
List<String> collectList = Stream.of(names).collect(Collectors.toList());
```

将Stream结果写入到集合类：

```java
String[] names = {"Alex", "Lion", "Linda", "Chandler", "Luck"};
LinkedList<String> collectionLinked = Stream.of(names
        ).collect(Collectors.toCollection(LinkedList::new));

```

写入数组：

```java
String[] names = {"Alex", "Lion", "Linda", "Chandler", "Luck"};
String[] arr = Stream.of(names).toArray(String[]::new);
```

写入Map

```java
String[] names = {"Alex", "Lion", "Linda", "Chandler", "Luck"};
Map<Character, List<String>> groupList = Stream.of(names).collect(
    Collectors.groupingBy(s -> s.charAt(0))
);
// 下面两种输出结果是一样的
for (Map.Entry<Character, List<String>> entry : groupList.entrySet()) {
    System.out.println(entry.getKey() + " - " + entry.getValue());
}

groupList.forEach((character, strings) -> System.out.println(character + " - " + strings));
```

## 计算结果

```java
String[] names = {"Alex", "Lion", "Linda", "Chandler", "Luck"};
// 计数
long count = Stream.of(names).count();
int[] nums = {1, 7, 3, 5, 8};
// 求和
int sum = IntStream.of(nums).sum();
int max = IntStream.of(nums).max().orElse(0);
// 大集合
IntSummaryStatistics statistics
                = IntStream.of(nums).summaryStatistics();
// IntSummaryStatistics{count=5, sum=24, min=1, average=4.800000, max=8}
System.out.println(statistics);
```

# 并行流

使用Stream还有另一个优点：方便并行处理。要使用并行流，在创建流时使用`parallelStream()`，或者在任意流加上`parallel()`。

```java
String[] names = {"Alex", "Lion", "Linda", "Chandler", "Luck"};
List<String> list = Arrays.asList(names);
// 创建并行流
Stream<String> stream = list.parallelStream().sorted();
stream.forEach(System.out::println);

// 使用parallel()
Stream<String> stream = Stream.of(names).parallel().sorted();
stream.forEach(System.out::println);
```

effective-java建议谨慎的使用并行流[48. 谨慎使用流并行](<http://sjsdfg.gitee.io/effective-java-3rd-chinese/#/notes/48.%20%E8%B0%A8%E6%85%8E%E4%BD%BF%E7%94%A8%E6%B5%81%E5%B9%B6%E8%A1%8C>)。**通常，并行性带来的性能收益在 ArrayList、HashMap、HashSet 和 ConcurrentHashMap 实例、数组、int 类型范围和 long 类型的范围的流上最好。**

# 相关资料

* [Java-Stream学习第一步：创建流](<http://www.kehaw.com/java/2019/11/14/Java-Stream%E5%AD%A6%E4%B9%A0-%E5%88%9B%E5%BB%BA%E6%B5%81.html>)
* [Java8中的流操作-基本使用&性能测试](<https://mp.weixin.qq.com/s?__biz=MzI4Njg5MDA5NA==&mid=2247485508&idx=2&sn=a686a128ccbcfa1fcc000d8b9de14155&chksm=ebd74945dca0c05378c3083c6efda294ea11db25705436d08a6d6af4e82993cac99804ee1553&token=2078489135&lang=zh_CN&scene=21#wechat_redirect>)
* [Java 8的Stream代码，你能看懂吗？](<https://mp.weixin.qq.com/s?__biz=MzI4Njg5MDA5NA==&mid=2247485026&idx=1&sn=8a99acd180aab1f5984f8b5eae8eab9f&chksm=ebd74763dca0ce758862de9453f155f9efdd28e39725b2067c54a5486449e8a14a1d5decb6c2&token=1755043505&lang=zh_CN&scene=21#wechat_redirect>)













