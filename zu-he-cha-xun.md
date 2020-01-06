---
description: Combined query
---

# 组合查询

## 组合查询

多少SQL查询都只包含从一个或多个表中返回数据的单条SELECT语句。MySQL也允许执行多个查询，并将结果作为单个查询结果集返回。这些组合查询通常为并或复合查询。

有两种基本情况，其中需要使用组合查询：

- 在单个查询中从不同的表返回类似结构的数据。
- 对单个表执行多个查询，按单个查询返回数据。

## 创建组合查询

可用UNION操作符来组合数条SQL查询。利用UNION，可给出多条SELECT语句，将它们的结果组合成单个结果集。

### 使用UNION

举个例子，假如需要价格小于等于5的所有物品的一个列表，而且还想包括供应商1001和1002生产的所有物品。当然，可用利用WHERE子句来完成工作，不过这次我们将使用UNION。

正如所述，创建UNION涉及编写多条SELECT语句。首先来看单条语句：

```mysql
SELECT vend_id, prod_id, prod_price
FROM products
WHERE prod_price <= 5;
```

```mysql
SELECT vend_id, prod_id, prod_price
FROM products
WHERE vend_id IN (1001,1002);
```

为了组合这两条语句，按如下进行：

```mysql
SELECT vend_id, prod_id, prod_price
FROM products
WHERE prod_price <= 5
UNION
SELECT vend_id, prod_id, prod_price
FROM products
WHERE vend_id IN (1001,1002);
```

作为参考，这里给出使用多条WHERE子句而不是使用UNION的相同查询：

```mysql
SELECT vend_id, prod_id, prod_price
FROM products
WHERE prod_price <= 5 
 OR vend_id IN (1001,1002);
```

在这个简单的例子中，使用UNION可能比使用WHERE子句更为复杂，但对于更复杂的过滤条件，或者从多个表中检索数据的情形，使用UNION可能会使处理更为简单。

### UNION规则

正如所见，并是非常容易使用的。但在进行并时有几条规则需要注意。

- UNION必须由两条或两条以上的SELECT语句组成，语句之间用关键字UNION分隔。
- UNION中的每个查询必须包含相同的列、表达式或聚集函数（不过各个列不需要以相同的次序列出）。
- 列数据类型必须兼容：类型不必完全相同，但必须是DBMS可以隐含地转换的类型。

### 包含或取消重复的行

UNION从查询结果集中自动去除了重复的行。这是UNION的默认行为，但是如果需要，可以改变它。事实上，如果想返回所有匹配行，可使用UNION ALL而不是UNION。

### 对组合查询结果排序

SELECT语句的输出用ORDER BY子句排序。在用UNION组合查询时，只能使用一条ORDER BY子句，它必须出现在最后一条SELECT语句之后。对于结果集，不存在用一种方式排序一部分，而又用另一种方式排序另一部分的情况，因此不允许使用多条ORDER BY子句。虽然ORDER BY子句似乎只是最后一条SELECT语句的组成部分，但实际上MySQL将用它来陪许所有SELECT语句返回的所有结果。