---
description: 'Insert,update,delete data'
---

# 插入、更新和删除数据

## 插入数据

毫无疑问，SELECT是最常使用的SQL语句了。但是，还有其他3个经常使用的SQL语句需要学习。

顾名思义，INSERT是用来插入（或添加）行到数据库表的。插入可以用几种方式使用：

- 插入完整的行。
- 插入行的一部分。
- 插入多行。
- 插入某些查询的结果。

## 插入完整的行

把数据插入表中的最简单的方法是使用基本的INSERT语法，它要求指定表名和被插入到新行中的值。

```mysql
INSERT INTO customers
VALUES(NULL,
			'Pep E. LaPew',
			'100 Main Street',
			'CA',
			'90046',
			'USA',
			NULL);
```

虽然这种语法很简单，但是并不安全，应该尽量避免使用。上面的SQL语句高度依赖于表中列的定义次序，并且还依赖于其次序容易获得的信息。即使可得到这种次序信息，也不能保证下一个表结构变动后各个列保持完全相同的次序。因此，编写依赖于特定列次序的SQL语句是很不安全的。如果这样做，有时难免会出问题。

编写INSERT语句的更安全的方法如下：

```mysql
INSERT INTO customers(cust_name,
		cust_address,
		cust_city,
		cust_state,
		cust_zip,
		cust_country,
		cust_contact,
		cust_email)
VALUES('Pep E. Lapew',
			...,
			这里就省略了)；
```

此例子完成与前一个INSERT语句完全相同的工作，但在表名后的括号里明确地给出了列名。在插入行时，MySQL将用VALUES列表中的相应值填入列表中的对应项。

因为提供了列名，VALUES必须以其指定的次序匹配指定的列名，不一定按哥哥列出现在实际表中的次序。其优点是，即使表的结构改变，此INSERT语句仍然能正常工作。

**总是使用列的列表。**一般不要使用没有明确给出列的列表的INSERT语句。使用列的列表能使SQL代码继续发挥作用。即使表结构发生了变化。

使用这种语法，还可以省略列。这表示可以只给某些列提供值，给其他列不提供值。

**省略列。**如果表的定义允许，则可以在INSERT操作中省略某些列。省略的列必须满足以下某个条件。

- 该列定义为允许NULL值（无值或空值）。
- 在表定义中给出默认值。这表示如果不给出值，将使用默认值。

如果对表中不允许NULL值且没有默认值的列不给出值，则MySQL将产生一条错误消息，并且相应的行插入不成功。

## 插入多个行

INSERT可以插入一行到一个表中。但如果你想插入多个行怎么办？可以使用多条INSERT语句，甚至一次提交它们，每条语句用一个分号结束。

```mysql
INSERT INTO customers(cust_name
		cust_address,
		cust_city,
		cust_state,
		cust_zip,
		cust_country)
VALUES(
			'Pep E. LaPew',
			'100 Main Street',
			'Los Angeles',
			'CA',
			'90046',
			'USA'),
			(
			'M Martian',
			'42 Galaxy Way',
			'New York',
			'NY',
			'11213',
			'USA'
			);
```

其中单条INSERT语句有多组值，每组值用一堆圆括号扩起来，用逗号分隔。

**提高INSERT的性能。**此技术可以提高数据看处理的性能，因为MySQL用单条INSERT语句处理多个插入比使用多条INSERT语句快。

## 插入检索出的数据

INSERT一般用来给表插入一个指定列值的行。但是，INSERT还存在另一种形式，可以利用它将一条SELECT语句的结果插入表中。这就是所谓的INSERT SELECT，顾名思义，它是由一条INSERT语句和一条SELECT语句组成的。

假如你想从另一表中合并客户列表到你的customers表中。不需要每次读取一行，然后再将它用INSERT插入，可以如下进行：

```mysql
INSERT INTO customers(cust_id,...,...)
SELECT cust_id,...,...
FROM custnew;
```

这个例子使用INSERT SELECT从custnew中将所有数据导入customers。

INSERT SELECT中的SELECT语句可包含WHERE子句以过滤插入的数据。

## 更新数据

为了更新（修改）表中的数据，可使用UPDATE语句。可采用两种方式使用UPDATE。

- 更新表中特定行。
- 更新表中所有行。

**不要省略WHERE子句。**在使用UPDATE时一定要注意细心，因为稍不注意，就会更新表中所有行。

UPDATE语句非常容易使用，甚至可以说是太容易使用了。基本的UPDATE语句由3部分组成，分别是：

- 要更新的表。
- 列名和它们的新值。
- 确定要更新行的过滤条件。

例如：

```mysql
UPDATE customers
SET cust_name = 'The Fudds',
		cust_email = 'elmer@fudd.com',
WHERE cust_id = 10005;
```

**IGNORE关键字。**如果用UPDATE语句更新多行，并且在更新这些行中的一行或多行时出现一个错误，则整个UPDATE操作被取消（错误发生前更新的所有行被恢复到它们原来的值）。即使是发生错误，也继续进行更新，可使用IGNORE关键字，如下所示：

```mysql
UPDATE IGNORE customers...
```

为了删除某个列的值，可设置它为NULL。

## 删除数据

为了从一个表中删除数据，使用DELETE语句。可以两种方式使用DELETE。

- 从表中删除特定的行。
- 从表中删除所有行。

例如：

```mysql
DELETE FROM customers
WHERE cust_id = 10006;
```

DELETE不需要列名或通配符。DELETE删除整行而不是删除列，为了删除指定的列，请使用UPDATE语句。

**更快的删除。**如果想从表中删除所有行，不要使用DELETE，可使用TRUNCATE TABLE语句，它完成相同的工作，但速度更快（TRUNCATE实际是删除原来的表并重新创建一个表，而不是逐行删除表中的数据）。