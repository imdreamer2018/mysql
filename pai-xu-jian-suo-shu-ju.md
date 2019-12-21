---
description: Sorting retrieved data
---

# 排序检索数据

## 排序数据

正如上一章所诉，下面的SQL语句返回某个数据库表的单个列，输出是没有特定的顺序的。其实，检索出的数据并不是以纯粹的随机顺序显示的，如果不排序，数据一般将以它在底层表中出现的顺序显示。关系数据库设计理论认为，如果不明确规定排序顺序，则不应该假定检索出的数据的顺序有意义。

**子句(clause)。**SQL语句由子句构成，有些子句是必需的，而有的是可选的。一个子句通常由一个关键字和所提供的数据组成。

为了明确的排序使用SELECT语句检索出的数据，可以使用ORDER BY子句。ORDER BY子句取一个或多个列的名字，据此对输出进行排序。

```mysql
SELECT prod_name
FROM products
ORDER BY prod_name;
```

这条语句除了指示MySQL对prod_name列以字母顺序排序数据的ORDER BY子句外，与前面的语句相同。

## 按多个列排序

为了按多个列排序，只要指定列名，列名之间用逗号分开即可。

```mysql
SELECT prod_id, prod_price, prod_name
FROM products
ORDER BY prod_price, prod_name
```

在按多个列排序时，排序完全安所规定的顺序进行。对于上述例子中的输出，仅在多个行具有相同的prod_price值时才对产品按prod_name进行排序。

## 指定排序方向

数据排序不限于升序排序，这只是默认的排序顺序，还可以使用ORDER BY子句以降序顺序排序。为了进行降序排序，必需指定DESC关键字。

```mysql
SELECT prod_id, prod_price, prod_name
FROM products
ORDER BY prod_price DESC;
```

如果打算用多个列排序，其中DESC关键字只应用直接位于其前面的列名，如果想再多个列上进行降序排序，必需对每个列指定DESC关键字。如下所示：

```mysql
SELECT prod_id, prod_price, prod_name
FROM products
ORDER BY prod_price DESC, prod_name
```

使用ORDER BY和LIMIT的组合，能够找出一个列中最高或最低的值。

```mysql
SELECT prod_price
FROM products
ORDER BY prod_price DESC
LIMIT 1;
```