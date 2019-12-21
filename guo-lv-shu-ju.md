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

## 结合WHERE子句

为了进行更强的过滤控制，MySQL允许给出多个WHERE子句。这些子句可用两种方式使用：以AND子句的方式或OR子句的方式使用。

### AND操作符

为了通过不止一个列进行过滤，可使用AND操作符给WHERE子句附加条件。

```mysql
SELECT prod_id, prod_price, prod_name
FROM products
WHERE vend_id = 1003 AND prod_price <= 10;
```

### OR操作符

OR操作符与AND操作符不同，它指示MySQL检索匹配任一条件的行。

```mysql
SELECT prod_name, prod_price
FROM products
WHERE vend_id = 1002 AND vend_id = 1003;
```

### 计算次序

WHERE可包含任意数目的AND和OR操作符。允许两者结合以进行复杂和高级的过滤。

假如需要列出价格为10以上且由1002或1003制造的所有产品。使用下面的语句：

```mysql
SELECT prod_name, prod_price
FROM products
WHERE vend_id = 1002 OR vend_id = 1003 AND prod_price >= 10;
```

但是上面的语句错误了，原因在于计算的次序。SQL在处理OR操作符前，优先处理AND操作符。当SQL看到上述WHERE子句时，它理解为由供应商1003制造的任何价格为10以上的产品，或者有供应商1002制造的任何产品，而不管其价格如何。所以，AND在计算次序中优先级更高。

为解决此问题，可使用圆括号明确的分组相应的操作符。

```mysql
SELECT prod_name, prod_price
FROM products
WHERE (vend_id = 1002 OR vend_id = 1003) AND prod_price >= 10;
```

因为圆括号具有较AND和OR操作符更高的计算次序，会首先购率圆括号内的OR条件。任何使用使用具有AND和OR操作符的WHERE子句，都应该使用圆括号明确的分组操作符，不要过分依赖默认的计算次序。使用圆括号没有什么坏处，它能消除歧义。

## IN操作符

圆括号在WHERE子句中还有另外一种用法。IN操作符用来指定条件范围，范围中的每个条件都可以进行匹配。IN取合法值的由逗号分隔的清单，全部括在圆括号中。

```mysql
SELECT prod_name, prod_price
FROM products
WHERE vend_id IN (1002,1003)
ORDER BY prod_name;
```

为什么要使用IN操作符？其优点如下：

- 在使用长的合法选项清单时，IN操作符的语法更清楚且更直观。
- 在使用IN时，计算的次序更容易管理。
- IN操作符一般比OR操作符清单执行更快。
- IN的最大优点是可以包含其他SELECT语句，是的能够更动态的建立WHERE子句。

## NOT操作符

WHERE子句中的NOT操作符有且只有一个功能，那就是否定它之后所跟的任何条件。

```mysql
SELECT prod_name, prod_price
FROM products
WHERE vend_id NOT IN (1002,1003)
ORDER BY prod_name;
```

为什么使用NOT？对于简单的WHERE子句，使用NOT确实没有什么优点，但是复杂的子句中，NOT是非常有用的。例如，在与IN操作符联合使用时，NOT使找出与条件列表不匹配的行非常简单。

**MySQL中的NOT，支持使用NOT对IN、BETWEEN和EXISTS子句取反。**

## LIKE操作符

前面介绍的所有操作符都是针对已知值进行过滤的。利用通配符可创建比较特定数据的搜索模式。在这个例子中，如果你想找到名称包含anvil的所有产品，可构造一个通配符搜索模式，找出产品名中任何未知出现anvil的产品。

**通配符(wildcard)**。用来匹配值的一部分的特殊字符。

**搜索模式(search pattern)**。有字面值、通配符或两者组合构成的搜索条件。

为在搜索子句中使用通配符，必需使用LIKE操作符。LIKE指示MySQL，后跟的搜索模式利用通配符匹配额入睡直接相等匹配进行操作。

### 百分号(%)通配符

最常使用的通配符是百分号(%)。在搜索串中，%表示**任何字符出现任意次数**。例如，为了找出所有以词jet起头的产品，可使用以下语句：

```mysql
SELECT prod_id, prod_name
FROM products
WHERE prod_name LIKE 'jet%';
```

**区分大小写**。根据MySQL的配置方式，搜索可以是区分大小写的。即'jet%'与JetPack 1000将不匹配。

通配符可在搜索模式中任意位置使用，并且可以使用多个通配符。

```mysql
SELECT prod_id, prod_name
FROM products
WHERE prod_name LIKE '%anvil%';
```

需要注意的是，除了一个或多个字符外，%还能匹配0个字符，带包搜索模式中给定未知的0个、一个或者多个字符。

**注意NULL**。虽然似乎%通配符可以匹配任何东西，但是有一个例外，即NULL，即使是WHERE prod_name LIKE '%' 也不能匹配用值NULL作为产品名的行。

### 下划线(_)通配符

另一个有用的通配符是下划线(_)。下划线的用途和%一样，但下划线只能匹配单个字符而不是多个字符。

```mysql
SELECT prod_id, prod_name
FROM products
WHERE prod_name LIKE '_ton anvil';
```

## 使用通配符的技巧

通配符搜索的处理一般要比前面讨论的其他搜索所花时间更长。这里给出一些使用通配符要记住的技巧。

- 不要过度使用通配符。如果其他操作符能达到相同的目的，应该使用其他操作符。
- 在确实需要使用通配符时，除非绝对有必要，否则不要把他们用在搜索模式的开始处。
- 仔细注意通配符的位置。如果放错地方，可能不会返回想要的数据。