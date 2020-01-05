---
description: Join table
---

# 联结表

## 联结 

SQL最强大的功能之一就是能在数据检索查询的执行中联结表。联结是利用SQL的SELECT能执行的最重要的操作，很好地理解联结及其语法是学习SQL的一个极为重要的组成部分。

### 关系表

 相同的数据出现多次不是一件好事，此因素是关系数据库设计的基础。关系表的设计就是要保证把信息分解成多个表，一类数据一个表。各表通过某些常用的值（ 即关系设计中的关系（relational））互相关联。

**外键（foreign key）**。外键为某个表中的一列，它包含另一个表的主键值，定义了两个表之间的关系。

这样做的好处如下：

- 供应商信息不重复，从而不浪费时间和空间。
- 如果供应商信息变动，可以只更新vendors表中的单个记录，相关表中的数据不用改动。
- 由于数据无重复，显然数据是一致的，这使得处理数据更简单。

总之，关系数据可以有效地存储和方便的处理。因此，关系数据库的可伸缩性远比非关系数据库要好。

**可伸缩性（scale）。**能够适应不断增加的工作量而不失败。设计良好的数据库或应用程序称之为可伸缩性好。

### 为什么要使用联结

正如所述，分解数据为多个表能更有效地存储，更方便地处理，并且具有更大的可伸缩性。但这些好处是有代价的。

如果数据存储在多个表中，怎样用单条SELECT语句检索出数据？

答案是使用联结。简单的说，联结是一种机制，用来在一条SELECT语句中关联表，因此称之为联结。使用特殊的语法，可以联结多个表返回一组输出，联结在运行时关联表中正确的行。

**维护引用完整性。**重要的是，要理解联结不是物理实体，换句话说，它在实际的数据库表中不存在。联结由MySQL根据需要建立，它存储于查询的执行当中。

## 创建联结

```mysql
SELECT vend_name, prod_name, prod_price
FROM vendors, products
WHERE vendors.vend_id = products.vend_id
ORDER BY vend_name, prod_name
```

这条语句的FROM子句列出了两个表，分别是vendors和products。它们就是这条SELECT语句联结的两个名字。这两个表用WHERE子句正确联结，WHERE子句指示MySQL匹配vendors表中vend_id和products表中的vend_id。可以看到要匹配的两个列以vendors.vend_id和products.vend_id指定。这里需要这种完全限定列明，因为如果只给出vend_id，则MySQL不知道指的是哪一个。                    

### WHERE子句的重要性                            

 利用WHERE子句建立联结关系似乎有点奇怪，但实际上，有一个很充分的理由。请记住，在一条SELECT语句中联结几个表时，相应的关系是运行中构造的。在数据库表的定义中不存在能指示MySQL如何对表进行联结的东西。在联结两个表时，你实际上做的是将第一个表中的每一行与第二个表中的每一个行匹配。WHERE子句作为过滤条件，它只包含那些匹配给定条件的行。没有WHERE子句，第一个表中的每个行将与第二个表中的每个行匹配，而不管它们逻辑上是否可以配在一起。

**笛卡尔积**。由沒有联结条件的表关系返回的结果为笛卡尔积。检索出的行的数目将是第一个表中的行数乘以第二个表中的行数。

### 内部联结

目前为止所用的联结称为等值联结，它基于两个表之间的相等测试。这种联结也称为内部联结。其实，对于这种联结可以使用稍微不同的语法来明确指定联结的类型。下面的SELECT语句返回与前面例子完全相同的数据：

```mysql
SELECT vend_name, prod_name, prod_price
FROM vendors INNER JOIN products
ON vendors.vend_id = products.vend_id
```

此语句中的SELECT与前面的SELECT语句相同，但FROM子句不同。这里，两个表之间的关系是FROM子句的组成部分，以INNER JOIN指定。在使用这种语法时，联结条件用特定的ON子句而不是WHERE子句给出。传递给ON的实际条件与传递给WHERE的相同。

**使用哪种语法**。ANSI SQL规范首选INNER JOIN语法。此外，尽管使用WHERE子句定义联结的确比较简单，但是使用明确的连接语法能够确保不会忘记联结条件，有时候这样做也能影响性能。

### 联结多个表

SQL对一条SELECT语句中可以联结的表的数目没有限制。创建联结的基本规则也相同。首先列出所有表，然后定义表之间的关系。

```mysql
SELECT prod_name, vend_name, prod_price, quantity
FROM orderitems, products, vendors
WHERE products.vend_id = vendors.vend_id
AND orderitems.prod_id = products.prod_id
AND order_num = 20005;
```

**性能考虑**。MySQL在运行时关联指定的每个表以处理联结，这种处理可能是非常耗费资源的。因此应该仔细，不要联结不必要的表，联结的表越多，性能下降越厉害。

回顾一下之前的例子。

```mysql
SELECT cust_name, cust_contact
FROM customers
WHERE cust_id IN (SELECT cust_id
				  FROM order
				  WHERE order_num IN (SEELCT order_num
				  					  FROM orderitems
				  					  WHERE prod_id = 'TNT2'));
```

子查询并不总是执行复杂SELECT操作的最有效的方法，下面是使用联结的相同查询。

```mysql
SELECT cust_name, cust_contact
FROM customers, order, orderitems
WHERE customers.cust_id = orders.cust_id
AND orderitems.order_num = orders.order_num
AND prod_id = 'TNT2';
```

联结是SQL中最重要最强大得特性，有效地使用联结需要对关系数据库设计有基本的了解。

## 使用表别名

上述介绍了如何使用别名引用被检索的表列。给列起别名的语法如下：

```mysql
SELECT Concat(RTrim(vend_name), '(', RTrim(vend_country), ')') AS vend_title
FROM vendors
ORDER BY vend_name;
```

别名除了用于列名和计算字段外，SQL还允许给表名起别名。这样有两个主要理由：

- 缩短SQL语句
- 允许在单条SELECT语句中多次使用相同的表

```mysql
SELECT cust_name, cust_contact
FROM customers AS c, orders AS o, orderitems AS oi
WHERE c.cust_id = o.cust_id
 AND oi.order_num = o.order_num
 AND prod_id = 'TNT2';
```

可以看到，FROM子句中的3个表全都具有别名，这使得能使用省写的c而不是全名customers。应该注意，表别名只在查询执行中使用，与列别名不一样，表别名不返回到客户机。

## 使用不同类型的联结

迄今为止，我们使用的只是称为内部联结或等值联结的简单联结。还有其他三种联结，分别是自联结、自然联结和外部联结。

### 自联结

如前所述，使用表别名的主要原因之一是能在单条SELECT语句中不止一次饮用相同的表。

例如，你发现某物品（ID为DTNTR）存在问题，因此想知道生产该物品的供应商生产的其他物品是否也存在这些问题。此查询要求首先找到生产ID为DTNTR的物品的供应商，然后找出这个供应商生产的其他物品。下面是解决此问题的一种方法：

```mysql
SELECT prod_id, prod_name
FROM products
WHERE vend_id = (SELECT vend_id 
								 FROM products
								 WHERE prod_id = 'DTNTR');
```

这是一种子查询的方法，现在来看使用联结的相同查询。

```mysql
SELECT p1.prod_id, p1.prod_name
FROM products AS p1, products AS p2
WHERE p1.vend_id = p2.vend_id
 AND p2.prod_id = 'DTNTR';
```

这里使用了表别名。products的第一次出现为别名p1，第二次出现为别名p2。现在可以将这些别名用作表名。例如，SELECT语句使用p1前缀明确地给出所需列的全名。如果不这样，MySQL将返回错误，因为分别存在两个名为prod_id，prod_name的列。MySQL不知道想要哪一个。WHERE通过匹配p1中的vend_id和p2中的vend_id首先联结两个表，然后按第二个表中的prod_id过滤数据，返回所需的数据。

**使用自联结而不用子查询。**自联结通常作为外部语句用来替代从相同表中检索数据时使用的子查询语句。虽然最终的结果是相同的，但有时候处理联结远比处理子查询快得多。应该试一试两种方法，以确定哪一种的性能更好。

### 自然联结

无论何时对表进行联结，应该至少有一个列出现在不止一个表中（被联结的表）。标准的联结（前一章中介绍的内部联结）返回所有数据，甚至相同的列多次出现。自然联结排除多次出现，使每个列只返回一次。

自然联结是这样一种联结，其中你只能选择那些唯一的列。这一般是通过对表使用通配符，对所有其他表的列使用明确的子集来完成的。下面举个例子：

```mysql
SELECT c.*, o.order_num, o.order_date,
			 oi.prod_id, oi.quantity, oi.item_price
FROM customers AS c, order AS o, orderitems AS oi
WHERE c.cust_id = o.cust_id
 AND oi.order_num = o.order_num
 AND prod_id = 'FB';
```

在这个例子中，通配符只对第一个表使用。所有其他列明确列出，所有没有重复的列被检索出来。

事实上，迄今为止我们建立的每个内部联结都是自然联结，很可能我们永远都不会用到不是自然联结的内部联结。

### 外部联结

许多联结将一个表中的行与另一个表中的行相关联。但有时候会需要包含没有关联行的那些行。例如，可能需要使用联结来完成一下工作：

- 对每个客户下了多少订单进行计数，包括那些至今尚未下订单的客户；
- 列出所有产品以及订购数量，包括没有人订购的产品；
- 计算平均销售规模，包括那些至今尚未下订单的客户；

在上述例子中，联结包含了那些在相关表中没有关联行的行。这种类型的联结称为外部联结。

下面的SELECT语句给出一个简单的内部联结。它检索所有客户及其订单：

```mysql
SELECT customers.cust_id, orders.order_num
FROM customers INNER JOIN orders
 ON customers.cust_id = orders.cust_id
```

外部联结语句类似，为了检索所有客户，包括那些没有订单的客户，可如下进行：

```mysql
SELECT customers.cust_id, orders.order_num
FROM customers LEFT OUTER JOIN orders
 ON customers.cust_id = orders.cust_id
```

类似于上一章中所看到的内部联结，这条SELECT语句使用了关键字**OUTER JOIN**来指定联结的类型（而不是在WHERE子句中指定）。但是，与内部联结关联连个表中的行不同的是，外部联结还包括没有关联行的行。在使用**OUTER JOIN**语法时，必须使用**RIGHT**或**LEFT**关键字指定包括其所有行的表（**RIGHT**指出的是OUTER JOIN右边的表，而**LEFT**指出的是OUTER JOIN左边的表）。上面的例子使用LEFT OUTER JOIN从FROM子句的左边表（customers表）中选择所有行。为了从右边的表中选择所有行，应该使用RIGHT OUTER JOIN，如下所示：

```mysql
SELECT customers.cust_id, orders.order_num
FROM customers RIGHT OUTER JOIN orders
 ON orders.cust_id = customers.cust_id;
```

## 使用带聚集函数的联结

聚集函数用来汇总数据。虽然至今为止聚集函数的所有例子只是从单个表汇总数据，但这些函数也可以与联结一起使用。

如果要检索所有客户及每个客户所下的订单数，下面使用了COUNT()函数的代码可完成此工作：

```mysql
SELECT customers.cust_name, customers.cust_id, COUNT(orders.order_num) AS num_ord
FROM customers INNER JOIN orders
 ON customers.cust_id = orders.cust_id
GROUP BY customers.cust_id;
```

此SELECT语句使用INNER JOIN将customers和orders表互相关联。GROUP BY子句按客户分组数据，因此，函数调用COUNT(orders.order_num)对每个客户的订单技术，将它作为num_ord返回。

当然聚集函数也可以方便的与其他联结一起使用。

## 使用联结和联结条件

在总结关于联结的这两章前，有必要汇总一下关于联结及其使用的某些要点。

- 注意所使用的联结类型。一般我们使用内部联结，但使用外部联结也是有效的。
- 保证使用正确的联结条件，否则将返回不正确的数据。
- 应该总是提供联结条件，否则会得出笛卡尔积。
- 在一个联结中可以包含多个表，甚至对于每个联结可以采用不同的联结类型。虽然这样做是合法的，一般也很有用，但应该在一起测试它们前，分别测试每个联结。这将使故障排除更为简单。

