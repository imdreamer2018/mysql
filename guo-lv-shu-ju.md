---
description: Filtering data
---

# 过滤数据

## 使用WHERE子句

数据库表一般包含大量的数据，很少需要检索表中所有行。通常只会根据特定操作或报告的需要提取表数据的子集。只检索所需数据需要指定搜索条件，搜索条件也成为过滤条件。

在SELECT语句中，数据根据WHERE子句中指定的搜索条件进行过滤。WHERE子句在表名(FROM)之后给出，如下所示：

```mysql
SELECT prod_name, prod_price
FROM products
WHERE prod_price = 2.50;
```

这个例子采用了简单的相等测试：它检测一个列是否具有指定的值，据此进行过滤。

**WHERE子句的位置**。在同时使用ORDER BY和WHERE子句时，应该让ORDER BY位于WHERE之后，否则将会产生错误。

## WHERE子句操作符

MySQL支持下表列出的所有条件操作符。

| 操作符  | 说明               |
| ------- | ------------------ |
| =       | 等于               |
| <>      | 不等于             |
| !=      | 不等于             |
| <       | 小于               |
| <=      | 小于等于           |
| >       | 大于               |
| >=      | 大于等于           |
| BETWEEN | 在指定的两个值之间 |

### 检查单个值

```mysql
SELECT prod_name, prod_price
FROM products
WHERE prod_name = 'fuses';
```

MySQL在执行匹配时默认不区分大小写。

```mysql
SELECT prod_name, prod_price
FROM products
WHERE prod_price < 10;
```

### 不匹配检查

```mysql
SELECT vend_id, prod_name
FROM products
WHERE vend_id <> 1003;
```

### 范围值检查

为了检查某个范围的值，可使用BETWEEN操作符，在使用BETWEEN时，必需指定两个所需范围的低端值和高端值，这两个值必需用AND关键字分隔。

```mysql
SELECT prod_name, prod_price
FROM products
WHERE prod_price BETWEEN 5 AND 10;
```

### 空值检查

在创建表时，表设计人员可以指定其中的列是否可以不包含值，一个列不包含值时，称其为包含空值NULL；

**NULL无值（no value）**，它与字段包含0，空字符串或仅仅包含空格不同。

SELECT语句有一个特殊的WHERE子句，可用来检查具有NULL值的列，这个WHERE子句就是IS NULL子句。

```mysql
SELECT prod_name
FROM products
WHERE prod_price IS NULL
```

**NULL与不匹配。**在通过过滤选择出不具有特定值的行时，你可能希望返回具有NULL值的行。但是不行，因为未知具有特殊的含义，数据库不知道它们是否匹配，所以在匹配过滤或不匹配过滤时不返回它们。因此，在过滤数据时，一定要验证返回数据中确实给出了被过滤列具有NULL的行。