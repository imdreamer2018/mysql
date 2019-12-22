---
description: Introduction to MySQL
---

# MySQL简介

## 什么是MySQL

数据的所有存储、检索、管理和处理实际上是由数据库软件——DBMS（数据库管理系统）完成的。MySQL是一种DBMS。由于MySQL的成本、性能、可信赖、简单，MySQL在世界范围内得到广泛使用。

### 客户机-服务器软件

DBMS可分为两类：一类为基于共享文件系统的DBMS，另一类为基于客户机-服务器的DBMS。

前者（包括诸如Microsoft Access和FIleMaker）用于桌面用途，通常不用于高端或更关键的应用。

MySQL、Oracle以及Microsoft SQL Server等数据库是基于客户机-服务器的数据库。服务器部分是负责所有数据访问和处理的一个软件。客户机是与用户打交道的软件。

* 服务器软件为MySQL DBMS。你可以在本地安装的副本上运行，也可以连接到运行在你具有访问权的远程服务器上的一个副本。
* 客户机可以是MySQL提供的工具、脚本语言、Web应用开发语言、程序设计语言等。

### MySQL版本

MySQL的当前版本为版本5。

## MySQL工具

如前所述，MySQL是一个客户机-服务器的DBMS，因此，为了使用MySQL，需要有一个客户机，即你需要用来与MySQL打交道的一个应用。

### mysql命令行使用程序

每个MySQL安装都有一个名为mysql的简单命令行应用程序。这个使用程序没有下拉菜单、流行的客户界面、鼠标支持或任何类似的东西。

**MySQL选项和参数。**如果仅输入mysql，可能会出现一个错误消息，因为可能需要安全证书，或者是因为MySQL没有运行在本地或默认端口上。例如为了指定用户登录名ben，应该使用mysql -u ben。为了给出用户名、主机名、端口和口令，应该使用

```text
mysql -u ben -p -h myserver -p 9999
```

### MySQL Administrator

MySQL Administrator是一个图形交互客户机，用来简化MySQL服务器的管理。

### MySQL Query Browser

MySQL Query Browser为一个图形交互客户机，用来编写和执行MySQL命令。

