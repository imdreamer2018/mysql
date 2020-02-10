---
description: Using view
---

# 使用视图

## 视图

视图是虚拟的表。与包含数据的表不一样，视图只包含使用时动态检索数据的查询。

理解视图的最好办法是看一个例子。

```mysql
SELECT cust_name, cust_contact
FROM customers, orders, orderitems,
WHERE customers.cust_id = orders.cust_id
	AND orderitems.order_num = order.order_num
	AND prod_id = 'TNT2';
```

现在，假如可以把整个查询包装成一个名为productcustomers的虚拟表，则可以如下轻松地检索出相同的数据：

```mysql
SELECT cust_name, cust_contact
FROM productcustomers
WHERE prod_id = 'TNT2';
```

这就是视图的作用。productcustomers是一个视图，作为视图，它不包含表中应该有的任何列或数据，它包含的是一个SQL查询。

### 为什么使用视图

下面是视图的一些常见应用：

- 重用SQL语句。
- 简化复杂的SQL操作。在编写查询后，可以方便地重用它而不必知道它的基本查询细节。
- 使用表的组成部分而不是整个表。
- 保护数据。可以给用户授予表的特定部分的访问权限而不是整个表的访问权限。
- 更改数据格式和表示。视图可返回与底层表的表示和格式不同的数据。

在视图创建之后，可以用与表基本相同的方式利用它们。可以对视图执行SELECT操作，过滤和排序数据，将视图联结到其他视图或表，甚至能添加和更新数据（添加和更新数据存在某些限制）。

重要的是知道视图仅仅是用来查看存储在别处的数据的一种设施。视图本身不包含数据，因此它们返回的数据是从其他表中检索出来的。在添加或更改这些表中的数据时，视图将返回改变过的数据。

**性能问题。**因为视图不包含数据，所以每次使用视图时，都必须处理查询执行时所需的任一个检索。如果你用多个联结和过滤创建复杂的视图或者嵌套了视图，可能会发现性能下降得很厉害。因此，在部署使用了大量视图的应用钱，应该进行测试。

### 视图的规则和限制

下面是关于视图创建和使用的一些最常见的规则和限制。

- 与表一样，视图必须唯一命名（不能给视图取与别的视图或表相同的名字）。
- 对于可以创建的视图数目没有限制。
- 为了创建视图，必须具有足够的访问权限。这些限制通常由数据库管理人员授予。
- 视图可以嵌套，即可以利用从其他视图中检索数据的查询来构造一个视图。
- ORDER BY可以用在视图中，但如果从该视图检索数据的SELECT语句中也包含ORDER BY，那么该视图中的ORDER BY将被覆盖。
- 视图不能索引，也不能有关联的触发器或默认值。
- 视图可以和表一起使用。

## 使用视图

在理解什么是视图（以及管理它们的规则和约束）后，我们来看一下视图的创建。

- 视图用CREATE VIEW语句来创建。
- 使用SHOW CREATE VIEW viewname；来查看创建视图的语句。
- 用DROP删除视图，其语法为DROP VIEW viewname;
- 更新视图时，可以先用DROP在用CREATE，也可以直接用CREATE OR REPLACE VIEW。如果要更新的视图不存在，则第二条更新语句会创建一个视图；如果要更新的视图存在，则第二条更新语句会替换原有视图。

### 利用视图简单复杂的联结

视图的最常见的应用之一是隐藏复杂的SQL，这通常都会设计联结。

```mysql
CREATE VIEW productcustomers AS
SELECT cust_name, cust_contact, prod_id
FROM customers, orders, orderitems
WHERE customers.cust_id = orders.cust_id
	AND orderitems.order_num = orders.order_num;
```

这条语句创建一个名为productcustomers的视图，它联结三个表，以返回已订购了任意产品的所有客户的列表。

为了检索订购了产品TNT2的客户，可如下进行：

```mysql
SELECT cust_name, cust_contact
FROM productcustomers
WHERE prod_id = 'TNT2';
```

这条语句通过WHERE子句从视图中检索特定数据。在MySQL处理此类查询时，它将制定的WHERE子句添加到视图查询中的已有WHERE子句中，以便正确过滤数据。

可以看出，视图极大地简化了复杂SQL语句的使用。利用视图，可一次性编写基础的SQL，然后根据需要多次使用。

**创建可重用的视图。**创建不受特定数据限制的视图是一种好办法。例如，上面创建的视图返回生产所有产品的客户而不仅仅是生产TNT2的客户。扩展视图的返回不仅使得它能被重用，而且甚至更有用。这样做不需要创建和维护多个类似视图。

### 用视图重新格式化检索出的数据

如上所述，视图的另一常见用途是重新格式化检索出的数据。

```mysql
SELECT Concat(RTrim(vend_name),'(', RTrim(vend_country),')') AS vend_title
FROM vendors
ORDER BY vend_name;
```

现在，假如经常需要这样格式的结果。不必在每次需要时执行联结，创建一个视图，每次需要时使用它即可。为把此语句转换为视图，可按如下进行：

```mysql
CREATE VIEW vendorlocations AS
SELECT Concat(RTrim(vend_name),'(', RTrim(vend_country),')') AS vend_title
FROM vendors
ORDER BY vend_name;
```

为此我们可以直接使用视图查询。

```mysql
SELECT *
FROM vendorlocations;
```

### 用视图过滤不想要的数据

视图对于应用普通的WHERE子句也很有用。例如，可以定义customeremaillist视图，它过滤没有电子邮件地址的客户。为此目的，可使用下面的语句：

```mysql
CREATE VIEW customeremaillist AS
SELECT cust_id, cust_name, cust_email
FROM customers
WHERE cust_email IS NOT NULL;
```

现在可以像使用其他表一样使用视图customeremaillist。

```mysql
SELECT *
FROM customersemaillist;
```

### 使用视图与计算字段

视图对于简化计算字段的使用特别有用。

```mysql
SELECT prod_id, quantity, item_price, quantity*item_price AS expanded_price
FROM orderitems
WHERE order_num = 20005;
```

将其转换为一个视图，如下进行：

```mysql
CREATE VIEW orderitemsexpanded AS
SELECT order_num, prod_id, quantity, item_price, quantity*item_price AS expanded_price
FROM orderitems;
```

### 更新视图

通常，视图是可更新的（即可以对它们使用INSERT、UPDATE和DELETE）。更新一个视图将更新其基表（可以回忆一下，视图本身是没有数据）。如果你对视图增加或删除行，实际上是对其基表增加或删除行。

但是，并非所有视图都是可更新的。基本上可以说，如果MySQL不能正确地确定被更新的基数据，则不允许更新（包括插入和删除）。这实际上意味着，如果视图定义有以下操作，则不能进行视图的更新：

- 分组（使用GROUP BY和HAVING）
- 联结
- 子查询
- 并
- 聚集函数（Min()、Count()、Sum()等）
- DISTINCT
- 导出（计算）列

**将视图用于检索**。一般，应该将视图用于检索（SELECT语句）而不用于更新（INSERT、UPDATE和DELETE）。