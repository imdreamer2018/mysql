---
description: Packet data
---

# 分组数据

## 数据分组

目前为止的所有计算都是在表的所有数据或匹配特定的WHERE子句的数据上进行的。下面的例子返回供应商1003提供的产品数目。

```mysql
SELECT COUNT(*) AS num_prods
FROM products
WHERE vend_id = 1003;
```

但如果要返回每个供应商提供的产品数目怎么办？这就需要分组了，分组允许把数据分为多个逻辑组，以便能对每个组进行聚集计算。

## 创建分组

分组是在SELECT语句的GROUP BY子句中建立的。下面的例子指定了两个列，vend_id包含产品供应商的ID，num_prods为计算字段。GROUP BY子句指示MySQL按vend_id排序并分组数据。这导致对每个vend_id而不是整个表计算num_prods一次。

```mysql
SELECT vend_id, COUNT(*) AS num_prods
FROM products
GROUP BY vend_id
```

因为使用了GROUP BY，就不必指定要计算和估值的每个组了。系统会自动完成。GROUP BY子句指示MySQL分组数据，然后对每个组而不是整个结果集进行聚集。

在具体使用GROUP BY子句前，需要知道一些重要的规定。

- GROUP BY子句可以包含任意数目的列。这使得能对分组进行嵌套，为数据分组提供更细致的控制。
- 如果在GROUP BY子句中嵌套了分组，数据将在最后规定的分组上进行汇总。换句话说，在建立分组时，指定的所有列都一起计算。
- GROUP BY子句中列出的每个列都必须是检索列或有效的表达式（但不能是聚集函数）。如果在SELECT中使用表达式，则必须在GROUP BY子句中指定相同的表达式。不能使用别名。
- 除聚集计算语句外，SELECT语句中的每个列都必须在GROUP BY子句中给出。
- 如果分组列中具有NULL值，则NULL将作为一个分组返回。如果列中有多行NULL值，它们将分为一组。
- GROUP BY子句必须出现在WHERE子句之后，ORDER BY子句之前。

**使用ROLLUP。**使用WITH ROLLUP关键字，可以得到每个分组以及每个分组汇总级别的值，如下所示。

```mysql
SELECT vend_id, COUNT(*) AS num_prods
FROM products
GROUP BY vend_id WITH ROLLUP;
```

## 过滤分组

除了能用GROUP BY分组数据外，MySQL还允许过滤分组，规定包括哪些分组，排除哪些分组。例如，可能想要列出至少两个订单的所有顾客。为得出这种数据，必须给予完整的分组而不是个别的行进行过滤。

我们已经看到了WHERE子句的作用，但是在这个例子中WHERE不能完成任务，因为WHERE过滤指定的是行而不是分组。事实上，WHERE没有分组的概念。

**HAVING支持所有WHERE操作符。**MySQL为此目的提供了另外的子句，那就是HAVING子句。HAVING非常类似于WHERE。事实上，目前为止所学过的所有类型的WHERE子句都可以用HAVING来替代。唯一的差别是WHERE过滤行，而HAVING过滤分组。

```mysql
SELECT cust_id, COUNT(*) AS orders
FROM orders
GROUP BY cust_id
HAVING COUNT(*) >= 2;
```

**HAVING和WHERE的差别。**这里另一种理解方法，WHERE在数据分组前进行过滤，HAVING在数据分组后进行过滤。这是一个重要的区别，WHERE排除的行不包括在分组中。这可能会改变计算值，从而影响HAVING子句中给予这些值过滤掉的分组。

那么，有没有在一条语句中同时使用WHERE和HAVING子句的需要呢？事实上，确实有。为了达到这一点，可增加一条WHERE子句，过滤订单价格为10以上的产品，然后在增加HAVING子句过滤出具有两个以上订单的分组，

```mysql
SELECT vend_id, COUNT(*) AS num_prods
FROM products
WHERE prod_price >= 10
GROUP BY vend_id
HAVING COUNT(*) >= 2;
```

## 分组和排序

虽然GROUP BY和ORDER BY经常完成相同的工作，但它们是非常不同的。下表汇总了它们之间的差别。

**ORDER BY和GROUP BY.**

| ORDER BY                                     | GROUP BY                                                 |
| -------------------------------------------- | -------------------------------------------------------- |
| 排序产生的输出                               | 分组行。但输出可能不是分组的顺序                         |
| 任意列都可以使用（甚至非选择的列也可以使用） | 只可能使用选择列或表达式列，而且必须使用每个选择列表达式 |
| 不一定需要                                   | 如果与聚集函数一起使用列（或表达式），则必须使用         |

为说明GROUP BY和ORDER BY的使用方法。下面这个例子，它检索总计订单价格大于等于50的订单的订单号和总计订单价格：

```mysql
SELECT order_num, SUM(quantity * item_price) AS ordertotal
FROM orderitems
GROUP BY order_num
HAVING SUM(quantity * item_price) >= 50;
```

为按总计订单价格排序输出，需添加ORDER BY子句。

```mysql
SELECT order_num, SUM(quantity * item_price) AS ordertotal
FROM orderitems
GROUP BY order_num
HAVING SUM(quantity * item_price) >= 50
ORDER BY ordertotal;
```

## SELECT子句顺序

下面回顾一下SELECT语句中子句的顺序。如下表所示。

**SELECT子句及其顺序。**

| 子句     | 说明               | 是否必须使用           |
| -------- | ------------------ | ---------------------- |
| SELECT   | 要返回的列或表达式 | 是                     |
| FROM     | 从中检索数据的表   | 仅在从表选择数据时使用 |
| WHERE    | 行级过滤           | 否                     |
| GROUP BY | 分组说明           | 尽在按组计算聚集时使用 |
| HAVING   | 组级过滤           | 否                     |
| ORDER BY | 输出排序顺序       | 否                     |
| LIMIT    | 要检索的行         | 否                     |

