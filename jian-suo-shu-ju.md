---
description: Retrieve data
---

# 检索数据

## SELECT语句

为了使用SELECT检索表数据，必须至少给出两条信息——想选择什么，以及从什么地方选择。

## 检索单个列

从简单的SQL SELECT语句开始介绍，此语句如下所示：

```mysql
SELECT prod_name FROM products;
```

上述语句利用SELECT语句从products表中检索一个名为prod_name的列。

**结束SQL语句。**   多条SQL语句必须以分号(;)分隔。

**SQL语句和大小写。**SQL语句不区分大小写，因此SELECT与select是相同的，许多SQL开发人员喜欢对所有SQL关键字使用大写，而对所有列和表名使用小写，这样做使代码更易于阅读和调试。

**使用空格**。在处理SQL语句时，其中所有空格都被忽略。多数SQL开发人员认为将SQL语句分成多行更容易阅读和调试。

## 检索多个列

下面的SELECT语句从products表中选择3列：

```mysql
SELECT prod_id, prod_name, prod_price
FROM products;
```

## 检索所有列

除了指定所需的列外，SELECT语句还可以检索所有的列而不必逐个列出它们。可以通过(*)通配符来达到。

```mysql
SELECT * 
FROM products;
```

## 检索不同的行

正如所见，SELECT返回所有匹配的行，但是如果你不想要每个值每次都出现，就可以使用DISTINCT关键字，此关键字指示MySQL只返回不同的值。DISTINCT关键字，它必须直接放在列名的前面。

```mysql
SELECT DISTINCT vend_id
FROM products;
```

**不能部分使用DISTINCT。**DISTINCT关键字应用于所有列而不仅是前置它的列。如果给出SELECT DISTINCT vend_id, prod_price，除非指定的两个列都不同，否则所有行都将被检索出来。

## 限制结果

SELECT语句返回所有匹配的行，它们可能是指定表中的每个行。为了返回第一行或前几行，可使用LIMIT子句。

```mysql
SELECT prod_name
FROM products
LIMIT 5;
```

LIMIT 5指示MySQL返回不多余5行。为得出下一个5行，可指定要检索的开始行和行数，如下所示：

```mysql
SELECT prod_name
FROM products
LIMIT 5,5;
```

## 使用完全限定的表名

迄今为止使用的SQL例子只通过列名引用列。也可能会使用完全限定的名字来引用列。

```mysql
SELECT products.prod_name
FROM products;
```

表名也可以是完全限定的，其中products表位于crashcourse数据库中，如下所示：

```mysql
SELECT products.prod_name
FROM crashcourse.products;
```

