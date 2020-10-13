---
title: 从Java到Go
date: 2020-03-26 16:11:58
tags:
categories:
---

趁着有点空闲时间，学习了一下Golang语言，作为Java语言的一种辅助。Go语言与Java有很多相似之处，当然也有它自己的特点，因此记录下Golang的一些特性，与使用了很久的Java做一个对比，便于更快更好的学习。

<!--more-->

# Hello World

```go
package main

import "fmt"

func main() {
	fmt.Println("Hello World")
}
```

* package是一个关键字，类似Java里的package
* main包是个特殊的包，表示当前是一个可执行程序
* import是一个关键字，和Java的import关键字一眼，引入后才可使用
* main函数一个主函数，表示程序的入口，和Java的main函数一样
* Println是fmt包的函数，和Java的System.out.println()一样

# 变量

# 常量

# 类型转换

# 访问权限

# 包

# 集合



## struct组合与继承

go使用组合来获得类似继承的功能。

```go
type Human struct {
	name string
	age  int
}

type Student struct {
	Human  // 匿名字段，默认包含Human的所有字段
	school string
}
func main() {
	jon := Student{
		Human:  Human{"Jon", 22},
		school: "NYU",
	}
	fmt.Printf("His name is %s\n", jon.name)
	fmt.Printf("His age is %d\n", jon.age)
	fmt.Printf("His school is %s\n", jon.school)
}
```

Student访问属性age的时候，有两种方式可以访问：通过直接访问age和访问Human作为字段名

```go
jon.age = 35
fmt.Printf("new age is %d\n", jon.age)
jon.Human.age = 29
fmt.Printf("new age is %d\n", jon.age)
```

如果Human和Student中含有相同的字段，访问时**最外层优先访问**，即bob.phone访问的是studnet字段

```go
type Human struct {
	name  string
	age   int
	phone string
}

type Student struct {
	Human  // 匿名字段，默认包含Human的所有字段
	school string
	phone  string
}

func main() {
	bob := Student{
		Human:  Human{"Bob", 23, "112233"},
		school: "NYU",
		phone:  "445566",
	}
	fmt.Printf("Bob's work phone is %s\n", bob.phone)//445566
	fmt.Printf("Bob's personal phone is %s", bob.Human.phone)//112233
}

```





# 函数与方法



## 方法

### method 继承

如果匿名字段实现了一个method，那么包含这个匿名字段的struct也能调用该methond。

### method重写

```go
// human定义方法
func (h *Human) SayHi() {
	fmt.Printf("Hi, I am %s , phone is %s \n", h.name, h.phone)
}

// employee 重写该方法
func (e *Employee) SayHi() {
	fmt.Printf("I am %s, work on %s\n", e.name, e.company)
}
```





# 指针

# 结构体



# 接口

# 面向对象





