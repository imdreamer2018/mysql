---
description: Full text search
---

# 全文本搜索

## 理解全文本搜索

前面介绍了LIKE关键字，它利用通配操作符匹配文本。使用LIKE，能够查找包含特殊值或部分值的行（不管这些值位于列内什么位置）。

还介绍了用基于文本的搜索作为正则表达式匹配列值的更进一步的介绍。使用正则表达式，可以编写查找所需行的非常复杂的匹配模式。

虽然这些搜索机制非常有用，但存在几个重要的限制。

**性能**——通配符和正则表达式匹配通常要求MySQL尝试匹配表中所有行（而且这些搜索极少使用表索引）。因此，由于被搜索行数不断增加，这些搜索可能非常耗时。

**明确控制**——使用通配符和正则表达式匹配，很难（而且并不总是能）明确的控制匹配什么和不匹配什么。例如，指定一个词必须匹配，一个词必须不匹配，而一个词仅在第一个词确实匹配的情况下才可以匹配或者才可以不匹配。

**智能化的结果**——虽然基于通配符和正则表达式的搜索提供了非常灵活的搜索，但它们都不能提供一种智能化的选择结果的方法。

所有这些限制以及更多的限制都可以用全文本搜索来解决。在使用全文本搜索时，MySQL不需要分别查看每个行，不需要分别分析和处理每个词。MySQL创建指定列中各词的一个索引，搜索可以针对这些词进行。这样，MySQL可以快速有效的决定哪些词匹配（哪些行包含它们），哪些词不匹配，它们匹配的频率，等等。

## 使用全文本搜索

为了进行全文本搜索，必须索引被搜索的列，而且要随着数据的改变不断的重新索引。在对表列进行适当设计后，MySQL会自动进行所有的所有和重新索引。

在索引之后，SELECT可与Match()和Against()一起使用以实际执行搜索。

### 启动全文本搜索支持

一般在创建表时启动全文本搜索。CREATE TABLE语句接收FULLTEXT子句，它给出被索引列的一个逗号分隔的列表。

下面的CREATE语句演示了FULLTEXT子句的使用：

```mysql
CREATE TABLE productnotes
(
	note_id		int				NOT NULL AUTO_INCREMENT,
	prod_id		char(10)	NOT NULL,
	note_date	datetime	NOT NULL,
	note_text	text			NULL,
	PRIMARY KEY(note_id),
	FULLTEXT(note_text)
) ENGINE=MyISAM;
```

这些列中有一个名为note_text的列，为了进行全文本搜索，MySQL根据子句FULLTEXT(note_text)的指示对它进行索引。这里的FULLTEXT索引单个列，如果需要也可以指定多个列。

在定义之后，MySQL自动维护该索引。在增加、更新或删除行时，索引随之自动更新。

可以在创建表时指定FULLTEXT，或者在稍后指定（在这种情况下所有已有数据必须立即索引）。

**不要在导入数据时使用FULLTEXT。**更新索引要花时间，虽然不是很多，但毕竟要花时间。如果正在导入数据到一个新表，此时不应该启用FULLTEXT索引。应该首先导入所有数据，然后再修改表，定义FULLTEXT。这样有助于更快的导入数据（而且使用索引数据的总时间小于在导入每行时分别进行索引所需的总时间）。

### 进行全文本搜索

在索引之后，使用两个函数Match()和Against()执行全文本搜索，其中Match()指定被搜索的行，Against()指定要使用的搜索表达式。

```mysql
SELECT note_text
FROM productnotes
WHERE Match(note_text) Against('rabbit');
```

此SELECT语句检索单个列note_text。由于WHERE子句，一个全文本搜索被执行。Match(note_text)指定MySQL针对指定的列进行搜索，Against('rabbit')指定词rabbit作为搜索文本。由于有两行包含词rabbit，这两个行被返回。

**使用完整的Match()说明。**传递给Match()的值必须与FULLTEXT()定义中的相同。如果指定多个列，则必须列出它们（而且次序正确）。

**搜索不区分大小写**。除非使用BINARY方式（本章中没有介绍），否则全文搜索不区分大小写。

事实是刚才的搜索可以简单地用LIKE子句完成，如下所示：

```mysql
SELECT note_text
FROM productnotes
WHERE note_text LIKE '%rabbit%'
```

### 使用查询扩展

查询扩展用来设法放宽所返回的全文本搜索结果的范围。考虑下面的情况。你想找出所有提到anvils的注释。只有一个注释包含词anvils，但你还想找出可能与你的搜索有关的所有其他行，即使它们不包含词anvils。

这也是查询扩展的一项任务。在使用查询扩展时，MySQL对数据和所有进行两遍扫面来完成搜索：

**首先**，进行一个基本的全文本搜索，找出与搜索条件匹配的所有行。

**其次**，MySQL检查这些匹配行并选择所有有用的词（我们将会简要的解释MySQL如何断定什么有用，什么无用）。

**再其次**，MySQL再次进行全文本搜索，这次不仅使用原来的条件，而且还使用所有有用的词。

利用查询扩展，能找出可能相关的结果，即使它们并不精确包含所查找的词。

下面举个例子，首先进行一个简单的全文本搜索，没有查询扩展：

```mysql
SELECT note_text
FROM productnotes
WHERE Match(note_text) Against('anvils');
```

这个只有一行包含词anvils，因此只返回一行。

下面是相同的搜索，这次使用查询扩展：

```mysql
SELECT note_text
FROM productnotes
WHERE Match(note_text) Against('anvils' WITH QUERY EXPANSION);
```

### 布尔文本搜索

MySQL支持全文搜索的另外一种形式，成为布尔方法。以布尔方式，可以提供关于如下内容的细节：

- 要匹配的词。
- 要排斥的词（如果某行包含这个词，则不返回该行，即使它包含其他指定的词也是如此）；
- 排列提示（指定某些词比其他词更重要，更重要的词等级更高）。
- 表达式分组。
- 另外一些内容。

**即使没有FULLTEXT索引也可以使用。**布尔方式不同于迄今为止使用的全文本搜索语法的地方在于，即使没有定义FULLTEXT索引，也可以使用它。但这是一种非常缓慢的操作（其性能将随着数据量的增加而降低）。

为演示IN BOOLEAN MODE的作用，举一个简单的例子：

```mysql
SELECT note_text
FROM productnotes
WHERE Match(note_text) Against('heavy' IN BOOLEAN MODE);
```

此文本搜索检索包含词heavy的所有行。其中使用了关键字IN BOOLEAN MODE，但实际上没有指定布尔操作符，因此，其结果与没有指定布尔方式的结果相同。

**IN BOOLEAN MODE的行为差异。**虽然这个例子的结果与没有IN BOOLEAN MDOE的相同，但其行为有一个重要的差别（即使再这个特殊的例子没有表现出来）。

为了匹配包含heavy但不包含任意以rope开始的词的行，可使用一下查询：

```mysql
SELECT note_text
FROM productnotes
WHERE Match(note_text) Against('heavy -rope*' IN BOOLEAN MODE);
```

这次只返回一行。这一次仍然匹配词heavy，但`-rope*`明确地只是MySQL排除包含`rope*`（任何以rope开始的词，包含ropes）的行，这就是为什么上一个例子中的第一行被排除的原因。

我们已经看到了两个全文本搜索布尔操作符`-`和`*`，`-`排除一个词，而`*`是截断操作符（可想象为用于词尾的一个通配符）。

**全文本布尔操作符**

| 布尔操作符 | 说明                                                         |
| ---------- | ------------------------------------------------------------ |
| +          | 包含，词必须存在                                             |
| -          | 排除，词必须不出现                                           |
| >          | 包含，而且增加等级值                                         |
| <          | 包含，且减少等级值                                           |
| ()         | 把词组成子表达式（允许这些子表达式作为一个组被包含、排除、排列等） |
| ～         | 取消一个词的排序值                                           |
| *          | 词尾的通配符                                                 |
| ""         | 定义一个短语（与单个词的列表不一样，它匹配整个短语以便包含或排除这个短语） |

下面举几个例子，说明某些操作符如何使用：

```mysql
SELECT note_text
FROM productnotes
WHERE Match(note_text) Against('+rabbit +bait' IN BOOLEAN MODE);
```

这个搜索匹配包含词rabbit和bait的行。

```mysql
SELECT note_text
FROM productnotes
WHERE Match(note_text) Against('rabbit bait' IN BOOLEAN MODE);
```

没有指定操作符，这个搜索匹配包含rabbit和bait中的至少一个词的行。

```mysql
SELECT note_text
FROM productnotes
WHERE Match(note_text) Against('"rabbit bait"' IN BOOLEAN MODE);
```

这个搜索匹配短语rabbit bait而不是匹配两个词rabbit和bait。

```mysql
SELECT note_text
FROM productnotes
WHERE Match(note_text) Against('>rabbit <carrot' IN BOOLEAN MODE);
```

匹配rabbit和carrot，增加前者的等级，降低后者的等级。

```mysql
SELECT note_text
FROM productnotes
WHERE Match(note_text) Against('+safe +(<combination)' IN BOOLEAN MODE);
```

这个搜索匹配词safe和combination，降低后者的等级

**排行而不排序。**在布尔方式中，不按等级值降序排序返回的行。

### 全文本搜索的使用说明

在结束之前，给出关于全文本搜索的某些重要的说明。

在索引全文本数据时，短词被忽略且从索引中排出。短词定义为那些具有3个或3个以下字符的词（如果需要，这个数目可以更改）。

MySQL带有一个内建的非用词（stopword）列表，这些词在索引全文本数据时总是被忽略。如果需要，可以覆盖这个列表（请参阅MySQL文档以了解如何完成此工作）。