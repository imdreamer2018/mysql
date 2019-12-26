---
description: Using subqueries
---

# 使用子查询

## 子查询

**查询（query）。**任何SQL语句都是查询。但此术语一般指SELECT语句。

SQL还允许创建子查询（subquery），即嵌套在其他查询中的查询。

## 利用子查询进行过滤

订单存储在两个表中，对于包含订单号、客户ID、订单日期的每个订单，orders表存储一行。各订单的物品存储在相关的orderitems表中。orders表不存储客户信息。它只存储客户的ID。实际的客户信息存储在customers表中。

现在假如需要列出订购物品TNT2的所有客户，应该怎样检索？

- 检索包含物品TNT2的所有订单的编号。
- 检索具有前一步骤列出的订单编号的所有客户的ID。
- 检索前一步骤返回的所有客户ID的客户信息。

上述每个步骤都可以单独作为一个查询来执行。可以把一条SEELCT语句返回的结果用于另一条SELECT语句的WHERE子句。也可以使用子查询来把3个查询组合成一条语句。

```mysql
SELECT order_num
FROM orderitems
WHERE prod_id = 'TNT2';
```

返回order_num有订单号为20005和20007。

```mysql
SELECT cust_id
FROM orders
WHERE order_num IN (20005,20007);
```

返回cust_id客户ID为10001和10004。

现在把第一个查询变成子查询组合两个查询。

```mysql
SELECT cust_id
FROM orders
WHERE order_num IN (SELECT order_num
										FROM orderitems
										WHERE prod_id = 'TNT2'
);
```

现在得到了订购物品TNT2的所有客户的ID。下一步是检索这些客户ID的客户信息。

```mysql
SELECT cust_name, cust_contact
FROM customers
WHERE cust_id IN(SELECT cust_id
								 FROM orders
								 WHERE order_num IN(SELECT order_num
								 										FROM orderitems
								 										WHERE prod_id = 'TNT2'
								 )
);
```

可见，在WHERE子句中使用子查询能够编写出功能很强并且很灵活的SQL语句。对于能嵌套的子查询的数目没有限制，不过在实际使用时由于性能的限制，不能嵌套太多的子查询。

虽然子查询一般与IN操作符结合使用，但也可以用于测试等于（=）、不等于（<>）等。

## 作为计算字段使用子查询

使用子查询的另一方法是创建计算字段。假如需要显示customers表中每个客户的订单总是。订单与相应的客户ID存储在orders表中。

为了执行这个操作，遵循下面的操作。

- 从customers表中检索客户列表。
- 对于检索出的每个客户，统计其在orders表中的订单数目。

```mysql
SELECT cust_name,cust_state,(SELECT COUNT(*)
														 FROM orders
														 WHERE orders.cust_id = customers.cust_id) AS orders
FROM customers
ORDER BY cust_name;
```

**相关子查询**。涉及外部查询的子查询。

这种类型的子查询成为相关子查询。任何时候只要列名可能有多义性，就必须使用这种语法（表名和列名由一个句点分隔）。

**逐渐增加子查询来建立查询。**用子查询测试和调试查询很有技巧性，特别是在这些语句的复杂性不断增加的情况下更是如此。用子查询建立（和测试）查询的最可靠的方法是逐渐进行，这与MySQL处理它们的方法非常相同。首先，建立和测试最内层的查询。然后，用硬编码数据建立和测试外层查询，并且尽在确认它正常后才嵌入子查询。这时，再次测试它。对于要增加的每个查询，重复这些步骤。这样做仅给构造查询增加了一点点时间，但节省了以后的大量时间，并且极大地提高了查询一开始就正常工作的可能性。