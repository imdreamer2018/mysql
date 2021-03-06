---
description: Using data data processing functions
---

# 使用数据处理函数

## 函数

与大多数计算机语言一样，SQL支持利用函数来处理数据。为了代码的可移植性，许多SQL程序员不赞成使用特殊实现的功能。虽然这样做很有好处，但不总是利于应用程序的性能。如果不使用这些函数，编写某些应用程序代码会很艰难。必须利用其他方法来实现DBMS非常有效的完成的工作。如果你决定使用函数，应该保证做好代码的注释。

## 使用函数

大多数SQL实现支持一下类型的函数。

- 用于处理文本串（如删除或填充值，转换值为大写或小写）的文本函数。
- 用于在数值数据上进行算术操作（如返回绝对值，进行袋鼠运算）的数值函数。
- 用于处理日期和时间值并从这些值中提取特定成分（例如，返回两个日期之差，检查日期有消息等）的日期和时间函数。
- 返回DBMS正使用的特殊信息（如返回用户登录信息，检查版本细节）的系统函数。

### 文本处理函数

常用的文本处理函数。

| 函数        | 说明              |
| ----------- | ----------------- |
| Left()      | 返回串左边的字符  |
| Length()    | 返回串的长度      |
| Locate()    | 找出串的一个子串  |
| Lower()     | 将串转换为小写    |
| LTrim()     | 去掉左边的空格    |
| RTrim()     | 去掉右边的空格    |
| Soundex()   | 返回串的SOUNDEX值 |
| SubString() | 返回子串的字符    |
| Upper()     | 将串转换为大写    |

上表中的SOUNDEX是一个将任何文本串转换为描述其语音表示的字母数字模式的算法。SOUNDEX考虑了类似的发音字符和音节，是的能对串进行发音比较而不是字母比较。

下面给出一个使用Soundex()函数的例子。customers表中有一个顾客Coyote Inc..其联系名为Y.Lee。但如果这是输入错误，此联系名实际应该是Y.Lie，怎么办？显然，按照常规的联系名搜索是不会返回数据的。我们可以使用Soundex()函数进行搜索，它匹配所有发音类似于Y.Lie的联系名：

```mysql
SELECT cust_name, cust_contact
FROM customers
WHERE Soundex(cust_contact) = Soundex('Y.Lie');
```

### 日期和时间处理函数

日期和时间采用相应的数据列下和特殊的格式存储，以便能快速和有效地排序和过滤，并节省物理存储空间。

**常用日期和时间处理函数**

| 函数          | 说明                           |
| ------------- | ------------------------------ |
| AddDate()     | 增加一个日期（天、周等）       |
| AddTime()     | 增加一个时间（时、分等）       |
| CurDate()     | 返回当前日期                   |
| CurTime()     | 返回当前时间                   |
| Date()        | 返回日期时间的日期部分         |
| DateDiff()    | 计算两个日期之差               |
| Date_Add()    | 高度灵活的日期运算函数         |
| Date_Format() | 返回一个格式化的日期或时间串   |
| Day()         | 返回一个日期的天数部分         |
| DayOfWeek()   | 对于一个日期，返回对应的星期几 |
| Hour()        | 返回一个时间的小时部分         |
| Minute()      | 返回一个时间的分钟部分         |
| Month()       | 返回一个日期的月份部分         |
| Now()         | 返回当前日期和时间             |
| Second()      | 返回一个时间的秒部分           |
| Time()        | 返回一个日期时间的时间部分     |
| Year()        | 返回一个日期的年份部分         |

首先需要注意的是MySQL使用的日期格式。无论你什么时候指定一个日期，不管是插入或更新表值还是用WHERE子句进行过滤，日期必须为格式yyyy-mm-dd。虽然MySQL支持2位数字的年份，但应该总是使用4位数字的年份。

```mysql
SELECT cust_id, order_num
FROM orders
WHERE order_date = '2005-09-01';
```

如果上述存储的order_date值为`2005-09-01 11:30:05`，则WHERE order_date = '2005-09-01'就不可以了。解决办法是指示MySQL仅将给出的日期与列中的日期部分进行比较，而不是将给出的日期与整个列值进行比较。为此，必须使用Date()函数。

```mysql
SELECT cust_id, order_num
FROM orders
WHERE Date(order_date) = '2005-09-01';
```

如果你想检索出2005年9月下的所有订单，可以使用下面几种解决办法。

```mysql
SELECT cust_id, order_num
FROM orders
WHERE Date(order_date) BETWEEN '2005-09-01' AND '2005-09-30';
```

还有一种不需要记住每个月有多少天或不需要操心闰年2月的办法。

```mysql
SELECT cust_id, order_num
FROM orders
WHERE Year(order_date) = 2005 AND Month(order_date) = 9;
```

### 数值处理函数

数值处理函数仅处理数值数据。这些函数一般主要用于代数、三角或几何运算。

**常用数值处理函数**

| 函数   | 说明               |
| ------ | ------------------ |
| Abs()  | 返回一个数的绝对值 |
| Cos()  | 返回一个角度的余弦 |
| Exp()  | 返回一个数的指数值 |
| Mod()  | 返回除操作的余数   |
| Pi()   | 返回圆周率         |
| Rand() | 返回一个随机数     |
| Sin()  | 返回一个角度的正弦 |
| Sqrt() | 返回一个数的平方根 |
| Tan()  | 返回一个角度的正切 |

## 聚合函数

我们经常需要汇总数据而不用把它们实际检索出来，为此MySQL提供了专门的函数。使用这些函数，MySQL查询可用于检索数据，以便分析和报表生成。这种类型的检索例子有以下几种。

- 确定表中行数（或者满足某个条件或包含某个特定值的行数）。
- 获得表中行组的和。
- 找出表列（或所有行或某些特定的行）的最大值、最小值和平均值。

因此返回实例表数据是对时间和处理资源的一种浪费。为了方便这种类型的检索，MySQL给出了5个聚集函数。

**聚集函数（aggregate function）。**运行在行组上，计算和返回单个值的函数。

| 函数    | 说明             |
| ------- | ---------------- |
| AVG()   | 返回某列的平均值 |
| COUNT() | 返回某列的行数   |
| MAX()   | 返回某列的最大值 |
| MIN()   | 返回某列的最小值 |
| SUM()   | 返回某列值之和   |

### AVG()函数

AVG()通过对表中行数计数并计算特定列值之和，求得该列的平均值。

**只用于单个列。**AVG()只能用来确定特定数值列的平均值，而且列名必须作为函数参数给出，为了获得多个列的平均值，必须使用多个AVG()函数。

**NULL值。**AVG()函数忽略列值NULL的行。

### COUNT()函数

可利用COUNT()确定表中行的数目或复合特定条件的行的数目。

COUNT()函数有两种使用方式。

- 使用COUNT(*)对表中行的数目进行计数，不管表列中包含的是NULL还是非空值。
- 使用COUNT(column)对特定列具有值的行进行计数，忽略NULL值。

**NULL值**。如果指定列名，则指定列的值为空的行被COUNT()函数忽略，但如果COUNT()函数中用的是星号（*），则不忽略。

### MAX()函数

MAX()返回指定列中的最大值。MAX()要求指定列名。

**对非数值数据使用MAX()**。虽然MAX()一般用来找出最大的数值或日期值，但MySQL允许将它用来返回任意列中的最大值，包括返回文本列中的最大值。在用于文本数据时，如果数据按相应的列排序，则MAX()返回最后一行。

**NULL值**。MAX()函数忽略列值为NULL的行。

### MIN()函数

MIN()函数正好与MAX()函数相反。

### SUM()函数

SUM()函数用来返回指定列值的和（总计）。

**在多个列上进行计算**。利用标准的算术操作符，所有聚集函数都可用来执行多个列上的计算。

**NULL值**。SUM()函数忽略列值为NULL的行。

## 聚集不同值

下面将要介绍的聚集函数的DISTINCT的使用，已经被添加到MySQL 5.0.3中。

以上5个聚集函数都可以如下使用：

- 对所有的行执行计算，指定ALL参数或不给参数（因为ALL是默认行为）
- 只包含不同的值，指定DISTINCT参数。

**ALL为默认。**ALL参数不需要指定，因为它是默认行为。如果不指定DISTINCT，则假定为ALL。

下面的例子使用AVG()函数返回特定提供商提供的产品的平均价格。它与上面的SELECT语句相同，但使用了DISTINCT参数，因此平均值只考虑各个不同的价格。

```mysql
SELECT AVG(DISTINCT prod_price) AS avg_price
FROM products
WHERE vend_id = 1003;
```

**注意**，如果指定列名，则DISTINCT只能用于COUNT()。DISTINCT不能用于COUNT(*)，因此不允许使用COUNT(DISTINCT)，否则会产生错误。类似的，DISTINCT必须使用列名，不能用于计算或表达式。

**将DISTINCT用于MIN()和MAX()。**虽然DISTINCT从技术上可用于MIN()和MAX()，但这样做实际上没有价值。一个列中的最小值和最大值不管是否包含不同值都是相同的。

## 组合聚集函数

目前为止的所有聚集函数例子都只涉及单个函数，但是实际上SELECT语句可根据需要包含多个聚集函数。