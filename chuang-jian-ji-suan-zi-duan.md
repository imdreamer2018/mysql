---
description: Create a calculated field
---

# 创建计算字段

## 计算字段

存储在数据库表中的数据一般不是应用程序所需要的格式。下面举几个例子。

- 如果想要在一个字段中既显示公司名字，又显示公司的地址，但这两个信息一般包含在不同的表列中。
- 城市、州和邮政编码存储在不同的列中，但是邮件标签打印程序却需要把它们作为一个恰当格式的字段检索出来。
- 列数据是大小写混合的，但报表程序需要把所有数据按大写表示出来。
- 物品订单表存储物品的价格和数量，但不需要存储每个物品的总价格。为打印发票，需要物品的总价格。
- 需要根据表数据进行总数、平均数计算或其他计算。

在上述每个例子中，存储在表中的数据都不是应用程序所需要的。我们需要直接从数据库中检索出转换、计算或格式化过的数据；而不是检索出数据，然后再在客户机应用程序或报告程序中重新格式化。

这就是计算字段发挥作用的所在了。与前面各章介绍过的列不同，计算字段并不实际存在于数据库表中。计算字段是运行时在SELECT语句内创建的。

## 拼接字段

为了说明如何使用计算字段，举一个创建由两列组成的标题的简单例子。

vendors表包含供应商店和位置信息。假如要生成一个供应商报表，需要在供应商的名字中按照name(location)这样的格式列出供应商的位置。

此报表需要单个值，而表中数据存储在两个列vend_name和vend_country中。此外，需要用括号将vend_country扩起来，这些东西都没有明确存储在数据库表中。

**拼接（concatenate）**。将值联结到一起构成单个值。

在MySQL的SELECT语句中，可使用Concat()函数来拼接两个列。其中Concat()把多个串连接起来形成一个较长的串。Concat()需要一个或多个指定的串，各个串之间用逗号分隔。

```mysql
SELECT Concat(vend_name,'(',vend_country,')')
FROM vendors
ORDER BY vend_name
```

**Trim函数**。MySQL除了支持RTrim()函数用于除去指定串右边的空格之外，还支持LTrim()函数除去指定串左边的空格。并且Trim()函数用于去掉两遍的空格。

### 使用别名

SQL支持列别名。别名(alias)是一个字段或值的替换名。别名用AS关键字赋予。

```mysql
SELECT Concat(RTrim(vend_name),'(',RTrim(vend_country),')') AS vend_title
FROM vendors
ORDER BY vend_name
```

这里的别名指示SQL创建一个包含指定计算的名为vend_title的计算字段，任何客户机应用都可以按名引用这个列，就像它是一个实际的表列一样。

## 执行算数计算

计算字段的另一场见用途是对检索出的数据进行算数计算。举个例子，orders表包含收到的所有订单，orderitems表包含每个订单中的各项物品，下面的SQL语句检索订单号20005中的所有物品。

```mysql
SELECT prod_id, quantity, item_price
FROM orderitems
WHERE order_num = 20005;
```

Item_price列包含订单中每项物品的单价。如下汇总物品的价格（单价乘以订购数量）。

```mysql
SELECT prod_id, quantity, item_price, quantity * item_price AS expanded_price
FROM orderitems
WHERE order_num = 20005;
```

MySQL支持加减乘除的基本算术操作符。此外，圆括号可用来区分优先顺序。