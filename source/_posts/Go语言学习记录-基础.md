---
title: Go语言学习记录-基础
date: 2019-12-15 20:22:27
tags: [Go]
categories: [Go]
---

学习go语言的记录

<!--more-->

[TOC]

# 基本结构和数据类型

## 基本结构和要素

```go
package main

import (
	"fmt"
)

func main()  {
	fmt.Println("hello")
}

```

每个go文件都属于且仅属于一个包。在源文件中非注释的第一行指明这个文件属于哪个包，如：`package main`。

对包重命名

```go
import fm "fmt"

```

### 函数

```go
func functionName(param1 type1, param2 type2...) 返回类型 {
    // 函数体
}
// 例如
func Sum(a, b int) int {/**/}

func add(a int, b string)
```

### 类型

```go
// 函数可有返回类型
func FuncName(a typea, b typeb) typeFunc
// 可有多个返回类型
func FuncName(a typea, b typeb) (t1 type1, t2 type2){
    return var1, var2
}
```

### 一般结构

Go程序启动顺序：

1. 按顺序导入所有被 main 包引用的其它包，然后在每个包中执行如下流程
2. 果该包又导入了其它的包，则从第一步开始递归执行，但是每个包只会被导入一次
3. 然后以相反的顺序在每个包中初始化常量和变量，如果该包含有 init 函数的话，则调用该函数
4. 在完成这一切之后，main 也执行同样的过程，最后调用 main 函数开始执行程序

### 输出 - 重要

```markdown
%s:
%d:十进制数字
%T: 类型
%v: 结构体字段

```



[fmt.Printf](<https://golang.org/pkg/fmt/>)

## 常量

使用关键字`const`定义

```go
const identifier [type] = value
const Pi = 3.14

// 显示定义
const B string = "abc"
// 隐式定义
const B = "abc"
```

数字型的常量是没有大小和符号的，并且可以使用任何精度而不会导致溢出

```go
const Ln2 = 0.693147180559945309417232121458
const Billion = 1e9
```

常量并行赋值

```go
const Monday, Tuesday, Wednesday, Thursday, Friday, Saturday = 1, 2, 3, 4, 5, 6

const (
	Monday, Tuesday, Wednesday = 1, 2, 3
	Thursday, Friday, Saturday = 4, 5, 6
)
```



```go
const(
	a = iota
	b
	c
)
```

第一个 `iota` 等于 0，每当 `iota` 在新的一行被使用时，它的值都会自动加 1；所以 `a=0, b=1, c=2`

## 变量

声明形式：var identifier type

```go
var startData int
```

当你在函数体内声明局部变量时，应使用简短声明语法 `:=`

```go
a := 1
```

并行赋值和交换

```go
a, b, c := 3, 5, "abc"

// 赋值交换
a, b := 1, 2
a, b = b, a
```



```go
package main

import "fmt"

func main() {
	a, b := 9, 3
	sum, sub := SumAndSub(a, b)
	fmt.Printf("res is : %d, %d", sum, sub)
}

func SumAndSub(a, b int) (int, int) {
	return a + b, a - b
}

```

### init函数



## 字符串和日期

默认使用UTF-8

两个字符串 `s1` 和 `s2` 可以通过 `s := s1 + s2` 拼接在一起

拼接的简写形式 `+=` 也可以用于字符串：

### strings和strconv



### 时间和日期

使用 time.Now()获取当前时间

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	t := time.Now()
	fmt.Println(t)
    
	// 2019-11-17
	fmt.Printf("%4d-%02d-%02d", t.Year(), t.Month(), t.Day())
}

```





## if-else

```go
if condition1 {
	// do something	
} else if condition2 {
	// do something else	
} else {
	// catch-all or default
}
```

在if中赋值。

```go
if val := 10; val > max {
	// do something
}
```

## switch

```go
switch var1 {
	case val1:
		...
	case val2:
		...
	default:
		...
}
```

变量 var1 可以是任何类型，而 val1 和 val2 则可以是同类型的任意值。类型不被局限于常量或整数，但必须是相同的类型.

如果在执行完每个分支的代码后，还希望继续执行后续分支的代码，可以使用 `fallthrough` 关键字来达到目的。

```go
switch i {
	case 0: fallthrough
	case 1:
		f() // 当 i == 0 时函数也会被调用
}
```

可选的 **`default`** 分支可以出现在任何顺序，但最好将它放在最后。它的作用类似与 `if-else` 语句中的 `else`，表示不符合任何已给出条件时，执行相关语句。

初始赋值

```go
switch result := calculate() {
	case result < 0:
		...
	case result > 0:
		...
	default:
		// 0
}
```

## for

```markdown
for 初始化语句; 条件语句; 修饰语句 {}
```

同时使用多个计数器

```go
for i, j := 0, N; i < j; i, j = i+1, j-1 {}
```

### for-range结构

```go
package main

import "fmt"

func main() {
	str := "Go is a beautiful language!"
	fmt.Printf("The length of str is: %d\n", len(str))
	for pos, char := range str {
		fmt.Printf("Character on position %d is: %c \n", pos, char)
	}
}
```



# 函数

```go
fucn 函数名(参数) (返回值){
    函数体
}
```



三中类型：

* 普通带有名字函数
* 匿名函数或lambda函数
* 方法

## 函数参数与返回值

基本类型是按值传递。切片（slice）、字典（map）、接口（interface）、通道（channel）这样的引用类型都是默认使用引用传递（即使没有显式的指出指针）

### 返回值有无命名

返回值有命名和无命名

```go
func getX2AndX3(x int) (int, int) {
	return 2 * x, 3 * x
}

func getX2AndX3_2(x int) (x2, x3 int) {
	x2 = 2 * x
	x3 = 3 * x
	return
}
```

**尽量使用命名返回值：会使代码更清晰、更简短，同时更加容易读懂。**

### 空白符

`ThreeValues` 是拥有三个返回值的不需要任何参数的函数，在下面的例子中，我们将第一个与第三个返回值赋给了 `i1` 与 `f1`。第二个返回值赋给了空白符 `_`，然后自动丢弃掉。

```go
func main() {
	var i1 int
	var i2 float32
	i1, _, i2 = ThreeValues()
	fmt.Printf("int: %d , float32: %f \n", i1, i2)
}

func ThreeValues() (int, int, float32) {
	return 5, 6, 7.9
}
```

## 变长参数

```go
func myFunc(a, b, arg ...int) {}
```

如果参数被存储在一个 slice 类型的变量 `slice` 中，则可以通过 `slice...` 的形式来传递参数，调用变参函数。

```go
func main() {
	x := min(5, 8, 2, 3)
	fmt.Printf("min is %d \n", x)
    // 如果参数被存储在一个 slice 类型的变量 slice 中，则可以通过 slice... 的形式来传递参数，调用变参函数。
	slice := []int{7, 9, 3, 5, 1}
	x = min(slice...)
	fmt.Printf("min is %d \n", x)
}

func min(s ...int) int {
	if len(s) == 0 {
		return 0
	}
	min := s[0]
	for _, value := range s {
		if value < min {
			min = value
		}
	}
	return min
}

```

## 匿名函数

函数作为返回值需要使用匿名函数。 匿名函数没有函数名。

```go
func (参数)(返回值){
    函数体
}
```

```go
// 将匿名函数保存到变量
sayHello := func() {
    print("Hello")
}
// 调用匿名函数
sayHello()

add := func(x, y int) {
    println(x + y)
}
add(2, 3)

// 在匿名函数后加上() 定义并执行
func(){
    println("Hello runing")
}()

func(x, y int) {
    println(x + y)
}(2, 3)
```

## 闭包

闭包之前的概念

```go
// 声明一个函数，函数名为a()，返回类型为 func
func a() func() {
	// 返回匿名函数
	return func() {
		println("Hello")
	}
}

func main() {
	res := a()
	// 相当于执行a函数内部的匿名函数
	res()
}
```

闭包

```go
// 声明一个函数，函数名为a()，返回类型为 func
func a() func() {
	name := "Chandler"
	// 返回匿名函数
	return func() {
		fmt.Println("Hello", name)
	}
}

func main() {
	// 闭包 = 函数 + 外层变量的引用
	// res此时就是一个闭包
	res := a()
	// 相当于执行a函数内部的匿名函数
	res()
}
```

将上面更改一种常见的闭包形式

```go
// 声明一个函数，函数名为a()，返回类型为 func
func a(name string) func() {
	// 返回匿名函数
	return func() {
		fmt.Println("Hello", name)
	}
}

func main() {
	// 闭包 = 函数 + 外层变量的引用
	// res此时就是一个闭包
	res := a("Ross")
	// 相当于执行a函数内部的匿名函数
	res()
}
```

闭包示例。使用闭包做文件后缀检测。

```go
// 返回类型为func(string) string
// 返回一个匿名函数，该匿名函数的返回类型为string
func makeSuffix(suffix string) func(string) string {
	return func(name string) string {
		// suffix是外层函数定义的变量
		if !strings.HasSuffix(name, suffix) {
			return name + suffix
		}
		return name
	}
}

func main() {
	// 闭包 = 函数 + 外层变量引用
	// jpgSuff包含了外层 suffix的引用
	jpgSuff := makeSuffix(".jpg")
	gifSuff := makeSuffix(".gif")
	fmt.Println(jpgSuff("demo")) //demo.jpg
	fmt.Println(gifSuff("demo")) //demo.gif
}

```

```go
func calc(base int) (func(int) int, func(int) int) {
	add := func(i int) int {
		base += i
		return base
	}

	sub := func(i int) int {
		base -= i
		return base
	}
	return add, sub
}

func main() {
	x, y := calc(100)
	res1 := x(50)     // base = 100 + 50
	fmt.Println(res1) // 150
	res2 := y(50)     // base = 150- 50
	fmt.Println(res2) // 100
}
```





**判断是不是闭包看他有没有引用外层函数变量。**



# 数组和切片

## 数组

```go
var arr [6]int //[5]int
var arr = new([5]int) //*[5]int
```

第一种变化

```go
var arr = [5]int{10, 20, 12, 22, 31}
```

第二种

```go
var arr = [...]int{5,6,7,8,9}
```

第三种

```go
var arr = [5]string{3: "Chris", 4: "Ron"}
```

只有索引 3 和 4 被赋予实际的值，其他元素都被设置为空的字符串，所以输出结果为

## 切片

切片（slice）是对数组一个连续片段的引用（该数组我们称之为相关数组，通常是匿名的），所以切片是一个引用类型。类似Python 中的 list 类型。

切片是一个 **长度可变的数组**。**优点** 因为切片是引用，所以它们不需要使用额外的内存并且比使用数组更有效率，所以在 Go 代码中 切片比数组更常用。

定义和声明

```go
// 不需要说明长度。切片在未初始化之前默认为 nil，长度为 0
var identifier []type
// 声明切片
var slice1 []type = arr[start:end]
// 定义
a := []int{3,6}
// 或者
a := [3]int{3,8,9}


// 基于数组定义
slice1 := 

```

### 基于数组定义

```go
a := [5]int{55, 56, 57, 58, 59}
b := arr[1:4]
```



`var slice1 []type = arr1[:]` 那么 slice1 就等于完整的 arr1 数组。或者``slice1 = &arr1`



### 切片传递给函数

```go
func main() {
	var arr = []int{2,3,4,5,6}
	fmt.Println(sum(arr))
}

func sum(a []int) int {
	s := 0
	for i := 0;i < len(a); i++ {
		s += i
	}
	return s
}
```

### 用make创建切片

```go
make([]type, size, cap)
```

* type：切片元素类型
* size：切片中元素数量
* cap：切片容量

len()返回size即元素数量。cap()返回容量

```go
var slice1 []type = make([]type, len)
slice1 := make([]type, len)
```

### 切片本质

切片的本质就是对底层数组的封装，它包含了三个信息：底层数组的指针、切片的长度（len）和切片的容量（cap）。

不能直接比较，只能和nil比较。判断是否为空使用len(slice) == 0

## map

定义语法：`map[keyType] valueType`

```go
func main() {
	score := make(map[string]int, 8)
	score["a"] = 90
	score["b"] = 80
	fmt.Print(score, "\n")
	fmt.Print(score["a"])
}
```

定义时填充

```go
func main() {
	info := map[string]string{
		"username": "root",
		"password": "1123",
	}
	fmt.Println(info)
}
```

常用方法

```go
func main() {
	info := map[string]string{
		"name": "root",
		"pwd": "1123",
	}
	fmt.Println(info)

	// 判断键是否存在
	if value, ok := info["name"]; ok {
		fmt.Println(value)
	}else {
		fmt.Println("no user")
	}

	// for range 遍历
	for k, v := range info {
		fmt.Println(k, ":", v)
	}
}

```

统计一段句子中每个单词出现的次数

```go
func main() {
	str := "how do you do"
	sp := strings.Split(str, " ")
	word := make(map[string] int, len(sp))

	for _,i := range sp{
		if _, ok := word[i]; ok {
			word[i] += 1
		}else {
			word[i] = 1
		}
	}
	fmt.Println(word)
}
```

map的排序

```go

```



# 结构体

## 类型别名与自定义

类型定义

```go
// 自定义类型是定义了一个全新的类型
type NewInt int
// 最后类型是 main.NewInt
```

类型别名

```go
type MyInt = int
// 最后类型是 int
```

## 结构体

```var
type 结构体实例 结构体类型
```



```go
type person struct {
	name, city string
	age        int8
}

func main() {
    // 实例化
	var Jon person
	Jon.name = "Jon Snow"
	Jon.age = 25
	Jon.city = "WinterFell"
	fmt.Println(Jon)
	fmt.Println("Jon's name is %s", Jon.name)
}

```

**匿名结构体**

```go
func main() {
	var user struct{name string; age int}
	user.name = "Rec"
	user.age = 29
	fmt.Printf("%#v \n", user)
}
```



1. 使用键值对初始化

```go
func main() {
	// 实例化
	user := person{
		name: "Kitty",
		city: "New York",
		age:  29,
	}
    // user is main.person{name:"Kitty", city:"New York", age:29}
	fmt.Printf("user is %#v", user)
}
```

1. 按照顺序提供初始化值。

```go
user := person{"Kitty", "New York", 19}
```

需要注意：必须初始化所有字段。填充顺序必须和声明顺序相同。不能和键值对声明混用

1. 使用 **new**， 分配指针 

```go
// user是一个结构体指针。支持直接使用 . 访问结构体成员
var user = new(person)
user.name = "Ross"
user.city = "New York"
user.age = 19

```

1. 取结构体地址实例化。相当于进行了一次new 操作

```go
user := &person{}
user.name = "Ross"
...
```



**构造函数**

该构造函数返回的是结构体指针类型

```go
func newPerson(name, city string, age int) *person {
	return &person{
		name: name,
		city: city,
		age:  age,
	}
}

func main() {
	Ross := newPerson("Ross", "New York", 28)
	fmt.Printf("%#v", Ross)
}
```

## 方法和接受者

**方法**是一种作用于特定类型变量的函数。叫做**接收者**，类似Java的**this**

```go
func (接收者变量 接收者类型) 方法名(参数列表) (返回列表){
    函数体
}
```

* 接收者变量：建议使用接收者类型的第一个小写字母，例如Person类型应该是p
* 接收者类型：类似参数，可以使指针类型和非指针类型

**方法与函数的区别是，函数不属于任何类型，方法属于特定的类型。**

```go
func newPerson(name, city string, age int) *person {
	return &person{
		name: name,
		city: city,
		age:  age,
	}
}

func (p person) Dream() {
	fmt.Printf("%s没有梦想！\n", p.name)
}

func main() {
	Ross := newPerson("Ross", "New York", 28)
	Ross.Dream()
}
```

**匿名字段**

结构体允许其成员字段在声明时没有字段名而只有类型，匿名字段默认采用类型名作为字段名，结构体要求字段名称必须唯一，因此一个结构体中同种类型的匿名字段只能有一个。



## 嵌套结构体

一个结构体中可以嵌套包含另一个结构体或结构体指针。

```go
type Address struct {
	Province string
	City     string
}

type User struct {
	Name    string
	Age     int8
	Address Address
}

func main() {
	user := User{
		Name: "Joey",
		Age:  27,
		Address: Address{
			Province: "江苏",
			City:     "南京",
		},
	}
	fmt.Printf("user = %#v \n", user)
}

```

## 继承

```go
type Animal struct {
	name string
}

func (a *Animal) move() {
	fmt.Printf("%s 能动 \n", a.name)
}

type Dog struct {
	Feet int8
	// 通过嵌套匿名结构体实现继承
	*Animal
}

func (d *Dog) wang() {
	fmt.Printf("%s 汪汪叫 \n", d.name)
}

func main() {
	d := Dog{
		Feet: 5,
		Animal: &Animal{
			name: "旺财",
		},
	}

	d.move()
	d.wang()
}

```



**结构体中字段大写开头表示可公开访问，小写表示私有（仅在定义当前结构体的包中可访问）。**



## 结构体与JSON

```go
type Student struct {
	ID   int
	Name string
}

type Class struct {
	Title    string
	Students []*Student
}

func main() {
	c := &Class{
		Title:    "2016",
		Students: make([]*Student, 0, 20),
	}

	for i := 0; i < 10; i++ {
		// 生成单个student
		stu := &Student{
			Name: fmt.Sprintf("stu%02d", i),
			ID:   i,
		}
		// 将生成的student添加到c.Students
		c.Students = append(c.Students, stu)
	}
	// JSON序列化，结构体 --> JSON格式字符串
	data, err := json.Marshal(c)
	if err != nil {
		fmt.Println("json marshal failed")
		return
	}
	fmt.Printf("json: %s \n", data)
}

```



# 接口

**接口是一种类型，抽象的类型。**

```go
type 接口类型名 interface{
    方法名1 (参数列表1) 返回值列表1
    方法名2 (参数列表2) 返回值列表2
    ...
}
```

接口名：使用`type`将接口定义为自定义的类型名。Go语言的接口在命名时，一般会在单词后面添加`er`

一个对象只要全部实现了接口中的方法，那么就实现了这个接口。换句话说，接口就是一个**需要实现的方法列表**。

定义**Sayer**接口

```go
// 定义Sayer接口
type Sayer interface {
	say()
}
```

定义cat和dog两个结构体

```go
type dog struct {
}

type cat struct {
}
```

给`dog`和`cat `分别实现`say`方法就可以实现`Sayer`接口

```go
func (d dog) say() {
	fmt.Println("汪汪汪")
}

func (c cat) say() {
	fmt.Println("喵喵喵")
}
```

接口类型变量能够存储所有实现了该接口的实例。如下，Sayer类型的变量能存储cat和dog类型的变量

```go
func main() {
	// 声明Sayer类型的变量x
	var s Sayer
	// 实例化一个cat和dog
	a := cat{}
	b := dog{}
	// cat实例直接赋值给s
	s = a
	s.say()
	s = b
	s.say()
}
```



## 接口嵌套

```go
type Mover interface {
	move()
}

type Sayer interface {
	say()
}

type animal interface {
	Mover
	Sayer
}

type cat struct {
	name string
}

func (c cat) move() {
	fmt.Printf("%s会动\n", c.name)
}

func (c cat) say() {
	fmt.Printf("%s喵喵叫\n", c.name)
}
```

## 空接口

没有定义任何方法的接口。任何类型都实现了空接口。空接口类型变量可以存储任意类型的变量



**空接口作为函数的参数**。接受任意类型的参数函数

```go
func show(a interface{}) {
	fmt.Printf("type:%T, value: %v\n", a, a)
}
```

**空接口作为map的值**。保存任意值

```go
var studentInfo = make(map[string]interface{})
studentInfo["name"] = "Joey"
studentInfo["age"] = 27
studentInfo["married"] = false
fmt.Println(studentInfo)
```

**类型断言**

一个接口的值是由**一个具体类型**和**具体类型的值**两部分组成。分别称为接口的动态类型和动态值

判断空接口中的值这个时候就可以使用类型断言，其语法格式

```go
x.(T)
```

* x：表示类型为**interface{}**的变量
* T：表示断言x可能是的类型

```go
func justifyType(x interface{}) {
	switch v := x.(type) {
	case string:
		fmt.Printf("x is a string, value is %v\n", v)
	case int:
		fmt.Printf("x is a int, value is %v\n", v)
	case bool:
		fmt.Printf("x is a bool, value is %v\n", v)
	default:
		fmt.Println("unknown type")
	}
}
```

当有两个或两个以上的具体类型必须以相同的方式进行处理时才需要定义接口。不要为了接口而写接口，那样只会增加不必要的抽象，导致不必要的运行时损耗。



# 反射

## reflect包

任意接口值在反射中都可以理解为由`reflect.Type`和`reflect.Value`两部分组成，并且reflect包提供了`reflect.TypeOf`和`reflect.ValueOf`两个函数来获取任意对象的Value和Type。

```go
func reflectType(x interface{}) {
	v := reflect.TypeOf(x)
	fmt.Printf("type: %v \n", v)
}
```





























