# Python-Sqlite3-Tutorial

## 1. 引言

SQLite 是一个由 C 语言编写的库，它提供了一个轻量级的基于磁盘的数据库，不需要单独开一个数据库服务器，并允许使用 SQL 语句访问数据库。

判断是否使用 SQLite 的标准：

- 数据与应用分离：不适用 SQLite
- 要求高并发性：不适用 SQLite
- 大数据：不适用 SQLite
- 否则，选择 SQLite

SQLite 不是一个 client/server 架构的数据库引擎（如 MySQL、Oracle、PostgreSQL 或 SQL Server 等），它致力于为单个应用程序提供本地存储，强调经济性、效率、可靠性、独立性和简单性，而不追求可伸缩性、并发性、集中性和控制性。

Python3 内置了一个名为 `sqlite3` 标准模块，提供了 SQLite 数据库操作的一整套接口，本文是利用该模块实现数据库操作的零基础入门教程。

> sqlite3 是以 [pysqlite](http://github.com/ghaering/pysqlite) 的名称在外部进行开发，其开发参考了 [PEP 249](https://www.python.org/dev/peps/pep-0249) 定义的 `Database API Specification 2.0`（数据库接口规范 2.0），以及 [SQLite](https://www.sqlite.org/) 官网提供的 SQL 语法和数据类型。

## 2. 目录

- [Python-Sqlite3-Tutorial](#python-sqlite3-tutorial)
  - [1. 引言](#1-引言)
  - [2. 目录](#2-目录)
  - [3. 安装](#3-安装)
  - [4. 教程](#4-教程)
    - [4.1. 创建和关闭数据库连接](#41-创建和关闭数据库连接)
    - [4.2. 新建表](#42-新建表)
    - [4.3. 插入记录【增】](#43-插入记录增)
    - [4.4. 查询记录【查】](#44-查询记录查)
    - [4.5. 修改记录【改】](#45-修改记录改)
    - [4.6. 删除记录【删】](#46-删除记录删)
    - [4.7. 删除表](#47-删除表)
  - [5. 附录](#5-附录)
  - [6. 概念](#6-概念)
    - [6.1. 数据类型](#61-数据类型)
    - [6.2. 字段定义](#62-字段定义)
  - [7. 接口](#7-接口)
    - [7.1. Connection](#71-connection)
    - [7.2. Cursor](#72-cursor)
  - [8. 参考](#8-参考)

## 3. 安装

`sqlite3` 是 Python3 的内置标准模块，不用额外安装。

## 4. 教程

这里简要介绍一下数据库操作的一般流程：

1. 开启数据库连接
2. 新建游标
3. 调用游标执行 SQL 语句
4. 关闭游标
5. 提交数据库更改（如果抛出异常，则回滚）
6. 关闭数据库连接

最常见的增删改查操作都通过执行 SQL 语句中实现。

### 4.1. 创建和关闭数据库连接

如要使用该模块，首先必须创建一个表示数据库的 `Connection` 对象。

```python
import sqlite3
conn = sqlite3.connect('demo.db')
conn.close()
```

如果不存在 `demo.db` 文件，将自动创建；如果存在，则直接进行连接。

### 4.2. 新建表

一旦你有了一个 `Connection`，你可以创建一个 `Cursor` 对象，并调用它的 `execute()` 方法来执行 SQL 命令，比如在数据库中新建一个名为 `student` 的表。

```python
import sqlite3
conn = sqlite3.connect('demo.db')
cursor = conn.cursor()
cursor.execute('''
    create table student(
        id integer primary key autoincrement not null,
        name text,
        age integer
    )
''')
conn.commit()
cursor.close()
conn.close()
```

如果表已经存在，会抛出 `OperationalError` 异常。为避免抛出该异常，可以使用 Python 的异常捕捉进行处理，也可以在 SQL 语句中进行表是否存在的判断。

```python
import sqlite3
conn = sqlite3.connect('demo.db')
cursor = conn.cursor()
cursor.execute('''
    create table if not exists student(
        id integer primary key autoincrement not null,
        name text,
        age integer
    )
''')
conn.commit()
cursor.close()
conn.close()
```

### 4.3. 插入记录【增】

接下来，让我们向表中插入几条记录用于测试。

- 插入一条记录

```python
import sqlite3
conn = sqlite3.connect('demo.db')
cursor = conn.cursor()
data = (1, 'john', 18)
sql = 'insert into student values (?, ?, ?)'
cursor.execute(sql, data)
conn.commit()
cursor.close()
conn.close()
```

- 插入多行记录

```python
import sqlite3
conn = sqlite3.connect('demo.db')
cursor = conn.cursor()
data = [
    (2, 'lily', 19),
    (3, 'mike', 20)
]
sql = 'insert into student values (?, ?, ?)'
cursor.executemany(sql, data)
conn.commit()
cursor.close()
conn.close()
```

通常，你不应该使用 Python 的字符串操作来组合 SQL 语句，因为这样做是不安全的，容易受到 SQL 注入攻击。正确的做法是使用 `?` 作为占位符，然后将值的元组或元组嵌套的列表作为第二个参数传递给 execute() 或 executemany() 方法。

### 4.4. 查询记录【查】

- 查询表中所有字段的记录

```python
import sqlite3
conn = sqlite3.connect('demo.db')
cursor = conn.cursor()
sql = 'select * from student'
cursor.execute(sql)
for item in cursor:
    print(item)
conn.commit()
cursor.close()
conn.close()
```

    (1, 'john', 18)
    (2, 'lily', 19)
    (3, 'mike', 20)

- 查询表中指定字段的记录

```python
import sqlite3
conn = sqlite3.connect('demo.db')
cursor = conn.cursor()
sql = 'select name from student'
cursor.execute(sql)
for item in cursor:
    print(item)
conn.commit()
cursor.close()
conn.close()
```

    ('john',)
    ('lily',)
    ('mike',)

- 根据条件查询表中的记录

```python
import sqlite3
conn = sqlite3.connect('demo.db')
cursor = conn.cursor()
sql = 'select * from student where age = 18'
cursor.execute(sql)
for item in cursor:
    print(item)
conn.commit()
cursor.close()
conn.close()
```

    (1, 'john', 18)

### 4.5. 修改记录【改】

例如，将 `mike` 的 `name` 修改为 `mary`。

```python
import sqlite3
conn = sqlite3.connect('demo.db')
cursor = conn.cursor()
sql = "update student set name = 'mary' where name = 'mike'"
cursor.execute(sql)
conn.commit()
cursor.close()
conn.close()
```

### 4.6. 删除记录【删】

- 按条件删除表记录

```python
import sqlite3
conn = sqlite3.connect('demo.db')
cursor = conn.cursor()
sql = "delete from student where name = 'lily'"
cursor.execute(sql)
conn.commit()
cursor.close()
conn.close()
```

- 清空表中所有记录

```python
import sqlite3
conn = sqlite3.connect('demo.db')
cursor = conn.cursor()
sql = 'delete from student'
cursor.execute(sql)
conn.commit()
cursor.close()
conn.close()
```

### 4.7. 删除表

```python
import sqlite3
conn = sqlite3.connect('demo.db')
cursor = conn.cursor()
sql = 'drop table student'
cursor.execute(sql)
conn.commit()
cursor.close()
conn.close()
```

## 5. 附录

数据库表操作常用的 SQL 命令：

| 功能                     | 命令                                                                                                                         |
| ------------------------ | ---------------------------------------------------------------------------------------------------------------------------- |
| 新建表                   | create table [if not exists] `TABLE_NAME` `COLUMN_DEF`                                                                       |
| 清空表中所有记录         | delete from `TABLE_NAME`                                                                                                     |
| 按条件清除表记录         | delete from `TABLE_NAME` where `EXPR`                                                                                        |
| 删除表                   | drop table [if exists] `TABLE_NAME`                                                                                          |
| 查看表字段               | pragma table_info(`TABLE_NAME`)                                                                                              |
| 修改表名称               | alter table `OLD_TABLE_NAME` rename to `NEW_TABLE_NAME`                                                                      |
| 添加表字段               | alter table `TABLE_NAME` add `NEW_COLUMN_NAME` [`COLUMN_DEF`]                                                                |
| 删除表字段               | alter table `TABLE_NAME` drop `COLUMN_NAME`                                                                                  |
| 修改字段名               | alter table `TABLE_NAME` rename column `OLD_COLUMN_NAME` to `NEW_COLUMN_NAME`                                                |
| 插入表记录               | insert into `TABLE_NAME` [(`COLUMN_NAME`)] values (`EXPR`)                                                                   |
| 向表中插入查询返回的数据 | insert into `TABLE_NAME` select `EXPR`                                                                                       |
| 在表末尾插入一条默认数据 | insert into `TABLE_NAME` default values                                                                                      |
| 更新表记录               | update `TABLE_NAME` set `COLUMN1_NAME` = `EXPR1` , `COLUMN2_NAME` = `EXPR2` [from `TABLE_NAME` or `SUBQUERY`] [where `EXPR`] |
| 查询表中所有字段的记录   | select \* from `TABLE_NAME` [where `EXPR`] [group by `EXPR`] [order by `ORDERING-TERM`] [limit `EXPR`]                       |
| 查询表中指定字段的记录   | select `COLUMN1_NAME` , `COLUMN2_NAME` from `TABLE_NAME`                                                                     |

更多关于 SQL 语句的详细信息请参见[官方文档](https://www.sqlite.org/doclist.html)。

## 6. 概念

### 6.1. 数据类型

SQLite 官方定义了如下五种[数据类型](https://www.sqlite.org/datatype3.html)：

- NULL：空值。
- INTEGER：整数。
- REAL：小数。
- TEXT：文本字符串。
- BLOB：二进制对象。

SQLite 没有单独的布尔数据存储类型，布尔值被存储为整数 0 (false) 和 1 (true)。

在 sqlite3 模块中，Python 数据类型和 SQLite 数据类型可以相互转换。

Python 数据类型在 SQLite 中的呈现方式：

| Python type        | SQLite type |
| ------------------ | ----------- |
| None               | NULL        |
| int                | INTEGER     |
| long               | INTEGER     |
| float              | REAL        |
| str (UTF8-encoded) | TEXT        |
| unicode            | TEXT        |
| buffer             | BLOB        |

SQLite 数据类型在 Python 中按照如下方式转换：

| SQLite type | Python type                                 |
| ----------- | ------------------------------------------- |
| NULL        | None                                        |
| INTEGER     | int or long, depending on size              |
| REAL        | float                                       |
| TEXT        | depends on text_factory, unicode by default |
| BLOB        | buffer                                      |

### 6.2. 字段定义

除数据类型外，定义字段时还有很多其他可选参数：

- primary key：主键。其值能唯一标识表中的每一行。
- primary key autoincrement：主键自增。插入数据时无需带上主键的值，因为它可以自动递增赋值。
- not null：非空。该字段内不允许出现空值。
- unique：唯一。该字段内不允许出现重复值。
- default：默认。设置该字段的默认值。

## 7. 接口

### 7.1. Connection

Connection 对象主要实现了以下方法：

- .close()：关闭数据库连接。关闭之后，试图对数据库进行的任何操作都将引发 `Error` 异常。
- .commit()：提交所有挂起的任务到数据库。
- .rollback()：回滚数据库到挂起任务尚未执行之前。值得注意的是，如果在没有提交更改的情况下就关闭了数据库连接，将会自动执行隐式回滚。
- .cursor()：返回一个新的使用该连接的 `Cursor` 对象。

### 7.2. Cursor

Cursor 对象主要实现了以下属性和方法：

- 属性
  - .description：游标描述信息。如果没有有意义的信息，就返回 `None`。
  - .rowcount：执行最后一个 SQL 语句操作的行数。如果没有 SQL 语句执行，返回 `-1`。
- 方法
  - .close()：关闭游标。关闭之后，试图对游标进行的任何操作都将引发 `Error` 异常。
  - .execute(sql [, parameters])：准备并执行 SQL 语句。
  - .executemany(sql, seq_of_parameters)：准备并执行 SQL 语句。接受的参数是一个序列。
  - .fetchone()：获取查询结果中的一行，返回单个序列。如果查询结果结果是空，返回 `None`。如果之前执行的 sql 语句没有任何返回结果，抛出 `Error` 异常。
  - .fetchmany([size=cursor.arraysize])：获取多行查询结果，返回一个嵌套的序列，例如，嵌套在列表里面的元组。
  - .fetchall()：获取查询结果的所有行，返回一个嵌套的序列。

## 8. 参考

> - [SQLite Home Page](https://www.sqlite.org/index.html)
> - [PEP 249 – Python Database API Specification v2.0](https://peps.python.org/pep-0249/)
> - [sqlite3 — DB-API 2.0 interface for SQLite databases](https://pysqlite.readthedocs.io/en/latest/sqlite3.html)
