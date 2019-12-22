---
description: Using the re to search.
---

# 用正则表达式进行搜索

## 正则表达式介绍

如前所述，过滤的例子允许用匹配、比较和通配操作符寻找数据。对于基本的过滤就足够了，但是随着过滤条件复杂性的增加，WHERE子句本身的复杂性也有必要增加。

这就需要正则表达式，正则表达式是用来匹配文本的特殊的串。所用种类的程序设计语言、文本编辑器、操作系统等都支持正则表达式。

## 使用MySQL正则表达式

正则表达式的作用是匹配文本，将一个模式与一个文本串进行比较。MySQL用WHERE子句对正则表达式提供了初步的支持，允许你指定正则表达式，过滤SELECT检索出的数据。

### 基本字符匹配

下面的语句检索列prod_name包含文本1000的所有行：

```mysql
SELECT prod_name
FROM products
WHERE prod_name REGEXP '1000'
ORDER BY prod_name;
```

除关键字LIKE被REGEXP替代外，这条语句看上去非常像使用LIKE的语句。REGEXP后所跟的东西为正则表达式处理。

为什么要费力的使用正则表达式？正则表达式确实没有带来太多好处，不过下面例子：

```mysql
SELECT prod_name
FROM products
WHERE prod_name REGEXP '.000'
ORDER BY prod_name;
```