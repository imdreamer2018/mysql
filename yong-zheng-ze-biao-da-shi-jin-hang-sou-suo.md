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

这里使用了正则表达式.000。.是正则表达式语言中一个特殊的字符。它表示匹配任意一个字符。

**LIKE和REGEXP**。它们之间有一个重要的差别。

```mysql
SELECT prod_name
FROM products
WHERE prod_name LIKE '1000'
ORDER BY prod_name;

SELECT prod_name
FROM products
WHERE prod_name REGEXP '1000'
ORDER BY prod_name;
```

如果执行上述两条语句，会发现第一条语句不返回数据，而第二条语句返回一行。

LIKE匹配整个行。如果被匹配的文本在列值中出现，LIKE将不会找到它，相应的行也不被返回（除非使用匹配符）。而REGEXP在列值内进行匹配，如果被匹配的文本在列值中出现，REGEXP将会找到它，相应的行将被返回。那么，REGEXP能匹配整个列值。

**匹配不区分大小写**。MySQL中的正则表达式不区分大小写。为区分大小写，可使用**BINARY**关键字，如WHERE prod_name REGEXP BINARY ‘JetPack .000‘。

### 进行OR匹配

为搜索两个串之一，使用|，|为正则表达式的OR操作符。如下所示：

```mysql
SELECT prod_name
FROM products
WHERE prod_name REGEXP '1000|2000'
ORDER BY prod_name;
```

### 匹配几个字符之一

如果只想匹配特定的字符，可通过指定一组用[和]扩起来的字符来完成，如下所示：

```mysql
SELECT prod_name
FROM products
WHERE prod_name REGEXP '[123] Ton'
ORDER BY prod_name;
```

这里使用了正则表达式[123]Ton。[123]定义一组字符，他的意思是匹配1或2或3，因此，1ton和2ton都匹配且返回。正如所见，[]是另一宗形式的OR语句。事实上，正则表达式[123]Ton为[1|2|3]Ton的缩写。

### 匹配范围

集合可用来定义要匹配的一个或多个字符，可以使用[123456789]集合来匹配数字0到9，为了简化这种类型的集合，可以使用-来定义一个范围，改成[0-9]。除此之外，[a-z]匹配任意字母字符。如下所示：

```mysql
SELECT prod_name
FROM products
WHERE prod_name REGEXP '[1-5]Ton'
ORDER BY prod_name;
```

这里使用正则表达式[1-5]Ton。

### 匹配特殊字符

正则表达式语言由具有特定含义的特殊字符构成。但是如果需要匹配类似|和-这种特殊字符，就必须用`\\`为前导，也就是转义字符。

```mysql
SELECT vend_name
FROM vendors
WHERE vend_name REGEXP '\\.'
ORDER BY vend_name
```

`\\`也用来引用元字符，如下表所示：

| 元字符 | 说明     |
| ------ | -------- |
| \\\f   | 换页     |
| \\\n   | 换行     |
| \\\r   | 回车     |
| \\\t   | 制表     |
| \\\v   | 纵向制表 |

当然，为了匹配反斜杠(\\)字符本身，需要使用`\\\`。

### 匹配字符类

存在找出你自己经常使用的数字、所有字母字符或数字字符相等的匹。为更方便工作，可以使用预定义的字符集，称为字符类。

| 类         | 说明                                                   |
| ---------- | ------------------------------------------------------ |
| [:alnum:]  | 任意字母和数字（同[a-zA-Z-0-9]）                       |
| [:alpha:]  | 任意字符（同[a-zA-Z]）                                 |
| [:blank:]  | 空格和制表（同[\\\t]）                                 |
| [:cntrl:]  | ASCII控制字符（ASCII 0到31和127）                      |
| [:digit:]  | 任意数字（同[0-9]）                                    |
| [:graph:]  | 与[:print:]相同，但不包括空格                          |
| [:lower:]  | 任意小写字母(同[a-z])                                  |
| [:print:]  | 任意可打印字符                                         |
| [:punct:]  | 既不在[:alnum:]又不在[:cntrl:]中的任意字符             |
| [:space:]  | 包括空格在内的任意空白字符（同[\\\f\\\n\\\r\\\t\\\v]） |
| [:upper:]  | 任意大写字母（同[A-Z]）                                |
| [:xdigit:] | 任意是十六进制数字（同[a-fA-F0-9]）                    |

### 匹配多个实例

下表列出正则表达式重复元字符。

| 元字符 | 说明                       |
| ------ | -------------------------- |
| *      | 0个或多个匹配              |
| +      | 1个或多个匹配              |
| ？     | 0个或者1个匹配             |
| {n}    | 指定数目的匹配             |
| {n,}   | 不少于指定数目的匹配       |
| {n,m}  | 匹配数目的范围(m不超过255) |

### 定位符

为了匹配特定位置的文本，需要使用下表列出的定位符。

| 元字符  | 说明       |
| ------- | ---------- |
| ^       | 文本的开始 |
| $       | 文本的结尾 |
| [[:<:]] | 词的开始   |
| [[:>:]] | 词的结尾   |

例如，如果你想要找出一个数（包括以小数点开始的数）开始的所有产品，怎么办？简单搜索[0-9\\\\.]（或[[:digit:]\\\\.]）不行，它将在文本内任意位置查找匹配，解决办法是使用^定位符。

```mysql
SELECT prod_name
FROM products
WHERE prod_name REGEXP '^[0-9\\.]'
ORDER BY prod_name;
```

**使用REGEXP类似LIKE的作用。**如前所述，LIKE和REGEXP的不同在于，LIKE匹配整个串而REGEXP匹配子串。利用定位符，通过用^开始每个表达式，用$结束每个表达式，可以是REGEXP的作用与LIKE一样。

**简单的正则表达式测试。**可以在不使用数据库表的情况下用SELECT来测试正则表达式。REGEXP检查总是返回0(没有匹配)或1(匹配)。可以用带文字串的REGEXP来测试表达式，并试验它们。相应的语法如下：

```mysql
SELECT 'hello' REGEXP '[0-9]'
```

这里显然会返回0，因为文本hello中没有数字。