---
title: MySQL基本语句总结
date: 2019-04-03 09:55:51
tags: MySQL
categories: MySQL
---

学习《MySQL必知必会》，整理基本语句

<!--more-->

## 查询
### 普通查询
#### DISTINCT 返回唯一的结果
```sql
SELECT DISTINCT id 
FROM products
```

#### LIMIT 限制查询结果
```SQL
SELECT id 
FROM products
LIMIT 5;
```

```
LIMIT 5,5
```
从5行开始的第五行。（检索的第一行为0为而不是1）

### 排序查询
#### ORDER BY
默认按照升序排序。

```SQL
SELECT id 
FROM products
ORDER BY age, name;
```
首先按照age，在按照name，仅age 有多个相同的时候。

##### 排序方向
```sql
ORDER BY age DESC, name
```
按照age降序排序。name 升序排序。

### 过滤数据

#### WHERE

在SELECT语句中，数据根据WHERE子句中指定的搜索条件进行过滤。 WHERE子句在表名（FROM子句）之后给出 。

WHERE子句的位置 在同时使用ORDER BY和WHERE子句时，应 该让ORDER BY位于WHERE之后， 否则将会产生错误  

```sql
SELECT id 
FROM products
WHERE id = 3;
```

##### BETWEEN

```sql
WHERE price BETWEEN 5 AND 10;
```
BETWEEN匹配范围中所有的值，包括指定的开始值和结束值。  

##### AND
```sql
WHERE price age < 10 AND price > 5;
```

##### OR

```sql
WHERE price age < 10 OR price > 5;
```

##### 计算次序

WHERE可包含任意数目的AND和OR操作符。允许两者结合以进行复杂 和高级的过滤。 

SQL（像多数语言一样）在处理OR操作符前，**优先处理AND操 作符**。AND在计算次序中优先级更高

```sql
WHERE (id = 1001 OR id = 1003) AND price > 10;
```
因为圆括号具有较AND或OR操作符高 的计算次序，DBMS首先过滤圆括号内的OR条件。
 任何时候使用具有AND和OR操作 符的WHERE子句，都应该使用圆括号明确地分组操作符。

##### IN

IN  WHERE子句中用来指定要匹配值的清单的关键字，功能与OR 相当。 

IN操作符用来指定条件范 围，范围中的每个条件都可以进行匹配。
```sql
WHERE id IN (1002, 1004);
```
 **IN操作符一般比OR操作符清单执行更快。** IN的最大优点是可以包含其他SELECT语句，使得能够更动态地建 立WHERE子句。  

### 使用别名

```sql
SELECT vend_name, '(',vend_country, ')' AS vend_title
```

使用**Concat**拼接字段

```mysql
SELECT
	CONCAT(vend_name,'(',vend_country, ')')
FROM
	vendors 
order by vend_name
```

执行算数计算



## 函数

### 聚集函数
* AVG()
* COUNT
* MAX
* MIN()
* SUM()

```sql
SELECT AVG(age) AS avg_price
```

## 分组

### GROUP BY

```sql
SELECT id, COUNT(*) AS num_prods
FROM products
GROUP BY id;
```
将分别计算每个id的数量。GROUP BY子句指 示MySQL按vend_id排序并分组数据。这导致对每个vend_id而不是整个表 计算num_prods一次 。

GROUP BY子句必须出现在WHERE子句之后， ORDER BY子句之前。  

### HAVING

大部分类型的WHERE子句都可以用HAVING来替代。唯一的差别是 WHERE过滤行，而HAVING过滤分组。 
HAVING支持所有WHERE操作符 
```sql
SELECT id, COUNT(*) AS orders
FROM orders
GROUP BY id
HAVING COUNT(*) >= 2;
```
HAVING子句，它过滤COUNT(*) >=2（两个以上的订单）的那些
分组。

有另一种理解方法，WHERE在数据 分组前进行过滤，HAVING在数据分组后进行过滤。这是一个重 要的区别，WHERE排除的行不包括在分组中

```sql
SELECT id, COUNT(*) AS orders
FROM orders
WHERE price >= 10
GROUP BY id
HAVING COUNT(*) >= 2;
```
第一行是使用了聚集函数的基本SELECT，它与前 面的例子很相像。WHERE子句过滤所有prod_price至少为10的 行。然后按vend_id分组数据，HAVING子句过滤计数为2或2以上的分组。 

### 分组和排序

一般在使用GROUP BY子句时，应该也给 出ORDER BY子句。这是保证数据正确排序的唯一方法。

```sql
SELECT order_num, SUM(quantity*item_price) AS ordertotal
FROM orderitems
GROUP BY order_num
HAVING SUM(quantity*item_price) >= 50
ORDER BY ordertotal
```

GROUP BY子句用来按订单号（order_num列） 分组数据，以便SUM(*)函数能够返回总计订单价格。HAVING子 句过滤数据，使得只返回总计订单价格大于等于50的订单。后，用ORDER BY子句排序输出。 

### 去重

**DISTINCT**和 **GROUP BY**都可用于去重，但是大表一般用distinct效率不高。

1. 当对系统的性能高并数据量大时使用group by
2. 当对系统的性能不高时使用数据量少时两者皆可
3. 尽量使用group by

## 子查询

```sql
SELECT order_num
FROM orderitems
WHERE prod_id = 'TNT2'
```

```sql
SELECT cust_id
FROM orders
WHERE order_num IN (20005, 20007)
```

```sql
SELECT cust_id
FROM orders
WHERE order_num IN (
	SELECT order_num
    FROM orderitems
    WHERE prod_id = 'TNT2'
)
```

首先执行括号中的查询

## 联结 Join

外键（foreign key）  外键为某个表中的一列，它包含另一个表 的主键值，定义了两个表之间的关系

### 创建联结

```sql
SELECT vent_name, prod_name, prod_price
FROM vendors, products
WHERE vendors.vend_id = products.vend_id
ORDER BY vend_name, prod_name
```

WHERE子句 指示MySQL匹配vendors表中的vend_id和products表中的vend_id。 

### 内部联结(INNER JOIN ON)

```sql
SELECT vend_name, prod_name, prod_price
FROM vendors INNER JOIN products
ON vendors.vend_id = products.vend_id
```

与前面语句相同。两个表之间的关系是FROM子句的组成部分，以INNER JOIN指定。联结条件用特定的ON子句而不是WHERE 子句给出。传递给ON的实际条件与传递给WHERE的相同。 

```mysql
SELECT
	cust_name,
	cust_contact 
FROM
	customers as c,
	orders as o,
	orderitems as oi 
WHERE
	c.cust_id = o.cust_id
	AND o.order_num = oi.order_num 
	AND prod_id = 'TNT2'
```

首选 INNER JOIN

## 高级联结

### 使用表别名

```sql
SELECT cust_name, cust_age
FROM customers AS c, orders AS o, ordertimes AS oi
```

建立c作为customers 的别名

### 自联结

```sql
SELECT
	p1.prod_id,
	p1.prod_name 
FROM
	products AS p1,
	products as p2
WHERE
	p1.vend_id = p2.vend_id
	AND p2.prod_id = 'DTNTR';
```

products是同一张表，第一次出现使用p1,p1前缀明确地给出所需列的全名；第二次出现使用p2

**使用自联结替代子查询**

### 自然联结

### 外部联结(LEFT OUTER JOIN)

联结包含了那些在相关表中没有关联行的行。这种 类型的联结称为外部联结  

```sql
SELECT
	customers.cust_id,
	orders.order_num 
FROM
	customers
	LEFT JOIN orders ON customers.cust_id = orders.cust_id
```

使用**LEFT OUTER JOIN**从FROM 子句的左边表（customers表）中选择所有行。



## 联结详解

### 内连接 INNER JOIN ON

![image](https://img-blog.csdn.net/20171209135846780?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcGxnMTc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

### 左连接 LEFT JOIN ON / LEFT OUTER JOIN ON

左边的表选择所有行，右边只显示符合搜索条件的记录

![image](F:\Hexo\source\_posts\MySQL基本语句总结\SouthEast)

### 右连接 RIGHT JOIN ON / RIGHT OUTRE JOIN ON

![](https://img-blog.csdn.net/20171209144056668?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcGxnMTc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)



## 组合查询

可用UNION操作符来组合数条SQL查询。利用UNION，可给出多条SELECT语句，将它们的结果组合成单个结果集。 

### UNION

```sql
SELECT
	vend_id,
	prod_id,
	prod_price 
FROM
	products 
WHERE
	prod_price <= 5 
UNION
SELECT
	vend_id,
	prod_id,
	prod_price 
FROM
	products 
WHERE
	vend_id IN ( 1001, 1002 )
```

等价于

```sql
SELECT vend_id, prod_id, prod_price
FROM products
WHERE prod_price <=5 OR vend_id IN (1001, 1002)
```

注意：

* UNION中的每个查询必须包含相同的列、表达式或聚集函数
* UNION必须由两条或两条以上的SELECT语句组成，语句之间用关 键字UNION分隔（因此，如果组合4条SELECT语句，将要使用3个 UNION关键字） 

### UNION ALL

UNION默认去除重复的行，使用UNION ALL 避免。

### 排序

SELECT语句的输出用ORDER BY子句排序。在用UNION组合查询时，只 能使用一条ORDER BY子句，它必须出现在最后一条SELECT语句之后。对 于结果集，不存在用一种方式排序一部分，而又用另一种方式排序另一 部分的情况，因此不允许使用多条ORDER BY子句。 

## 增改删

### 插入数据

```sql
INSERT INTO customers(cust_name,
                      cust_id
)
VALUES('name',
      'id'),
      ('name_2',
      'id_2');
```

### 更新数据

```SQL
UPDATE customers
SET cust_email = 'mail'
WHERE cust_id = 1001;
```

**注意不要忘了where**

### 删除数据

```SQL
DELETE FROM customers
WHERE cust_di = 1002;
```

## 表操作

### 创建表

```sql
CREATE TABLE customers(
    cust_id		int		NOT NULL AUTO_INCREMENT,
    cust_name	char(50)NOT NULL,
    PRIVARY KEY (cust_id)
)ENGINE=InnoDB;
```

## 参考

《MySQL必知必会》

[CSDN-图解MySQL 内连接、外连接、左连接、右连接、全连接……太多了](<https://blog.csdn.net/plg17/article/details/78758593>)