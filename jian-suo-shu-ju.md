---
description: Retrieve data
---

# 检索数据

## SELECT语句

为了使用SELECT检索表数据，必须至少给出两条信息——想选择什么，以及从什么地方选择。

## 检索单个列

从简单的SQL SELECT语句开始介绍，此语句如下所示：

```mysql
SELECT prod_name FROM products;
```

上述语句利用SELECT语句从products表中检索一个名为prod_name的列。

**结束SQL语句。**   多条SQL语句必须以分号(;)分隔。

**SQL语句和大小写。**SQL语句不区分大小写，因此SELECT与select是相同的，许多SQL开发人员喜欢对所有SQL关键字使用大写，而对所有列和表名使用小写，这样做使代码更易于阅读和调试。



