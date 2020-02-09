---
description: Create and operate table
---

# 创建和操纵表

## 创建表

MySQL不仅用于表数据操纵，而且还可以用来执行数据库和表的所有操作，包括表本身的创建和处理。

一般有两种创建表的方法：

- 使用具有交互式创建和管理表的工具
- 表也可以直接用MySQL语句操纵

为了用程序创建表，可使用SQL的CREATE TABLE语句。值得注意的是，在使用交互式工具时，实际上使用的是MySQL语句。但是，这些语句不是用户编写的，界面工具会自动生成并执行相应的MySQL语句。

### 表创建基础

为利用CREATE TABLE创建表，必须给出下列信息：

- 新表的名字，在关键字CREATE TABLE之后给出
- 表列的名字和定义，用逗号分隔

CREATE TABLE语句也可能会包含其他关键字或选项，但至少要包含表的名字和列的细节。下面的MySQL语句创建本书中所用的customers表：

```mysql
CREATE TABLE customers
(
	cust_id 				int				NOT NULL AUTO_INCREMENT,
	cust_name				char(50)	NOT NULL,
	cust_address		char(50)	NULL,
	cust_city				char(50)	NULL,
	...
	PRIMARY KEY (cust_id),
)ENGINE=InnoDB;
```

**语句格式化**。可回忆一下，以前说过MySQL语句中忽略空格，语句可以在一个长行上输入，也可以分成许多行。它们的作用都相同。这允许你以最适合自己的方法安排语句的格式。

**处理现有的表**。在创建新表时，指定的标明必须不存在，否则将出错。如果要防止意外覆盖已有的表，SQL要求首先手工删除该表，然后再创建它，而不是简单地用创建表语句覆盖它。如果你仅想在一个表不存在时创建它，应该在表名后给出IF NOT EXITSTS。这样做不检查已有表的模式是否与你打算创建的表模式相匹配。它只是查看表名是否存在，并且尽在表名不存在时创建它。

### 使用NULL值

NULL值就是没有值或缺值。允许NULL值的列也允许在插入行时不给出该列的值。不允许NULL值的列不接受该列没有值的行，换句话说，在插入或更新行时，该列必须有值。

**理解NULL**。不要把NULL值与空串相混淆。NULL值时没有值，它不是空串。如果指定‘’（两个单引号，其间没有字符），这在NOT NULL列中是允许的。空串是一个有效的值，它不是无值。NULL值用关键字NULL而不是空串指定。

### 主键再介绍

正如所说，主键值必须唯一。即，表中的每个行必须具有唯一的主键值。如果主键使用单个列，则它的值必须唯一。如果使用多个列，则这些列的组合值必须唯一。

为创建由多个列组成的主键，应该以逗号分隔的列表给出各个列，如下所示：

```mysql
CREATE TABLE orderitems
(
	order_num					int					NOT NULL,
	order_item				int					NOT NULL,
	prod_id						char(10)		NOT NULL,
	...
	PRIMARY KEY (order_nu, order_item)
)ENGINE=InnoDB;
```

### 使用AUTO_INCREMENT

AUTO_INCREMENT告诉MySQL，本列每当增加一行时自动增量。每次执行一个INSERT操作时，MySQL自动对该列增量（从而才有这个关键字AUTO_INCREMENT），给该列赋予下一个可用的值。这样给每个行分配一个唯一的cust_id，从而可以用作主键。

每个表只允许一个AUTO_INCREMENT列，而且它必须被索引。

### 指定默认值

如果在插入行时没有给出值，MySQL允许指定此时使用的默认值。默认值用CREATE TABLE语句的列定义中的DEFAULT关键字指定。

**不允许函数。**与大多数DBMS不一样，MySQL不允许使用函数作为默认值，它只支持常量。

**使用默认值而不是NULL值。**许多数据库开发人员使用默认值而不是NULL列，特别是对用于计算或数据分组的列更是如此。

### 引擎类型

与其他DBMS一样，MySQL有一个具体管理和处理数据的内部引擎。在你使用CREATE TABLE语句时，该引擎具体创建表，而在你使用SELECT语句或进行其他数据库处理时，该引擎在内部处理你的请求。多数时候，此引擎都隐藏在DBMS内，不需要过多关注它。

为什么要发行多种引擎呢？因为它们具有各自不同的功能和特性，为不同的任务选择正确的引擎能获得良好的功能和灵活性。

以下是几个需要知道的引擎：

- InnoDB是一个可靠的事物处理引擎，它不支持全文搜索；
- MEMORY在功能等同于MyISAM，但由于数据存储在内存中，速度很快；
- MyISAM是一个性能极高的引擎，它支持全文吧搜索，但不支持事务处理；

## 更新表

为更新表定义，可使用ALTER TABLE语句。但是，理想状态下，当表中存储数据以后，该表就不应该再被更新。在表的设计过程中需要花费大量时间来考虑，以便后期不对该表进行大的改动。

为了使用ALTER TABLE更该表结构，必须给出下面的信息：

- 在ALTER TABLE之后给出要更改的表名（该表必须存在，否则将出错）；
- 所做更改的列表；

下面的例子给表添加一个列：

```mysql
ALTER TABLE vendors
ADD vend_phone CHAR(20);
```

删除刚刚添加的列，可以这样做：

```mysql
ALTER TABLE vendors
DROP COLUNM vend_phone;
```

ALTER TABLE的一种常见用途是定义外键。下面是用来定义本书中的表所用的外键的代码：

```mysql
ALTER TABLE orderitems
ADD COSNTRANT fk_orderitems_orders
FOREIGN KEY (order_num) REFERENCES order (order_num);
```

复杂的表结构更改一般需要手动删除过程，它设计一下步骤：

用新的列布局创建一个新表；

使用INSERT SELECT语句从旧表复制数据到新表。如果有必要，可使用转换函数和计算字段；

- 检验包含所需数据的新表；
- 重命名旧表（如果确定，可以删除它）；
- 用旧表原来的名字重命名新表；
- 根据需要，重新创建触发器、存储过程、索引和外键。

## 删除表

删除表非常简单，使用DROP TABLE语句即可：

```mysql
DROP TABLE customers2;
```

## 重命名表

使用RENAME TABLE语句可以重命名一个表：

```mysql
RENAME TABLE customers2 TO customers;\
```