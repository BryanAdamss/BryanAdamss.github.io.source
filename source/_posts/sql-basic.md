---
title: sql-basic
tags:
  - sql
  - mysql
categories:
  - 其他
date: 2019-10-16 21:30:13
---

# SQL基础

## 数据库表组成
- 库db
    - 表table
        - 行(记录)1
            - 列(字段,field)1，列(字段,field)2
        - 行(记录)2
            - 列(字段,field)1，列(字段,field)2

## 注意项

### 大小写不敏感
- SQL对大小写不敏感
    - `SELECT`和`select`是一样的
    
### SQL执行顺序
- SQL语句执行的时候是有一定顺序的。
    - from 先选择一个表，构成一个结果集。
    - where 对结果集进行筛选，筛选出需要的信息形成新的结果集。
    - group by 对新的结果集分组。
    - having 筛选出想要的分组。
    - select 选择列。
    - order by 当所有的条件都弄完了。最后排序。
- 当一条查询语句中包含所有的子句，执行顺序依下列子句次序：
    - `FROM 子句`：执行顺序为从后往前、从右到左。数据量较少的表尽量放在后面。
    - `WHERE子句`：执行顺序为自下而上、从右到左。将能过滤掉最大数量记录的条件写在WHERE 子句的最右。
    - `GROUP BY`：执行顺序从左往右分组，最好在GROUP BY前使用WHERE将不需要的记录在GROUP BY之前过滤掉。
    - `HAVING 子句`：消耗资源。尽量避免使用，HAVING 会在检索出所有记录之后才对结果集进行过滤，需要排序等操作。
    - `SELECT子句`：少用`*`号，尽量取字段名称。ORACLE 在解析的过程中, 通过查询数据字典将`*`号依次转换成所有的列名, 消耗时间。
    - `ORDER BY子句`：执行顺序为从左到右排序，消耗资源。 

## 常用语句

### SELECT(选取数据)
- `SELECT`语句用于从数据库中选取数据。结果被存储在一个结果表中，称为结果集
```SQL
SELECT column_name,column_name FROM table_name;

/* 选取所有列*/
SELECT * FROM table_name;
```

### SELECT DISTINCT(去除列中重复值)
- 在表中，一个列可能会包含多个重复值，有时您也许希望仅仅列出不同（distinct）的值。DISTINCT 关键词用于返回唯一不同的值(去掉列的重复值)
```SQL
SELECT DISTINCT column_name,column_name FROM table_name;

SELECT DISTINCT country FROM Websites;
```

### WHERE子句(类似if)
- `WHERE`子句用于过滤记录
```SQL
SELECT column_name,column_name FROM table_name WHERE column_name operator value;

/*从Websites表中查出所有country为'CN'的记录(行)*/
SELECT * FROM Websites WHERE country='CN';
```
- 常用operator(操作符)

| 操作符  | 描述                               |
| ------- | ---------------------------------- |
| =       | 等于                               |
| <>、!=  | 不等于                             |
| >       | 大于                               |
| <       | 小于                               |
| >=      | 大于等于                           |
| <=      | 小于等于                           |
| BETWEEN | 在某个范围                         |
| LIKE    | 搜索某种模式(用于正则匹配)         |
| IN      | 指定针对某个列的多个可能值(枚举值) |

### AND&OR运算符(逻辑运算)
- 逻辑运算符，基于多个条件对记录进行过滤
```SQL
/*country为CN并alexa大于50*/
SELECT * FROM Websites WHERE country='CN' AND alexa > 50;

/*alexa大于15并且country为CN或USA*/
SELECT * FROM Websites WHERE alexa > 15 AND (country='CN' OR country='USA');
```

### ORDER BY(排序)
- ORDER BY 关键字用于对结果集进行排序。
```SQL
SELECT column_name,column_name FROM table_name ORDER BY column_name,column_name ASC | DESC;
```
- 顺序
    - ASC
        - 升序(默认)
    - DESC
        - 降序
- 多列排序
```SQL
/*结果集，先按country升序（默认），再按alexa升序（默认）*/
SELECT * FROM Websites ORDER BY country,alexa;

/*结果集，先按country降序，再按alexa升序（默认）*/
SELECT * FROM Websites ORDER BY country DESC,alexa;
```

### INSERT INTO(插入记录)
- INSERT INTO 语句用于向表中插入新记录。
```SQL

/*第一种形式无需指定要插入数据的列名，只需提供被插入的值即可：*/
INSERT INTO table_name
VALUES (value1,value2,value3,...);

/*第二种形式需要指定列名及被插入的值：*/
INSERT INTO table_name (column1,column2,column3,...)
VALUES (value1,value2,value3,...);
```

### UPDATE(更新记录)
- UPDATE 语句用于更新表中的记录。
```SQL
UPDATE table_name SET column1=value1,column2=value2,...WHERE some_column=some_value;
```
- 注意
    - **`WHERE 子句`规定哪条记录或者哪些记录需要更新。如果您省略了 `WHERE 子句`，所有的记录都将被更新！**
    - 在 MySQL 中可以通过设置 sql_safe_updates 这个自带的参数来解决，当该参数开启的情况下，你必须在update 语句后携带 where 条件，否则就会报错。set sql_safe_updates=1; 表示开启该参数
    
### DELETE(删除记录)
- DELETE 语句用于删除表中的行。
```SQL
DELETE FROM table_name WHERE some_column=some_value;
```
- **WHERE 子句规定哪条记录或者哪些记录需要删除。如果您省略了 WHERE 子句，所有的记录都将被删除！**

### LIMIT、TOP、ROWNUM(规定返回的记录数目)
-  MySQL 支持 LIMIT 语句来选取指定的条数数据， Oracle 可以使用 ROWNUM 来选取。
```SQL

SELECT column_name(s) FROM table_name LIMIT number;

/*从Persons中取出记录的前5条*/
SELECT * FROM Persons LIMIT 5;
```

### LIKE(搜索指定模式)
- LIKE 操作符用于在 WHERE 子句中搜索列中的指定模式。
- 类似win中搜索时用的搜索模式(类似*.png)
```SQL

SELECT column_name(s) FROM table_nameWHERE column_name LIKE pattern;

/*
%在SQL中代表任意数量的任意字符(任意通配符)，类似*的作用 
下面语句代表搜索所有name为G开头的记录
*/
SELECT * FROM Websites WHERE name LIKE 'G%';

/*k结尾*/
SELECT * FROM Websites WHERE name LIKE '%k';

/*中间包含oo*/
SELECT * FROM Websites WHERE name LIKE '%oo%';

/*中间不包含oo*/
SELECT * FROM Websites WHERE name NOT LIKE '%oo%';
```
- 常用通配符

| 通配符                   | 描述                                      |
| ------------------------ | ----------------------------------------- |
| %                        | 替代 0 个或多个字符                       |
| _                        | 替代一个字符                              |
| [charlist]               | 字符列中的任何单一字符(类比正则[a-zA-Z])  |
| [^charlist]或[!charlist] | 不在字符列中的任何单一字符(类比[^a-zA-Z]) |
 
 ### RLIKE、REGEXP(正则匹配)
 - MySQL 中使用 REGEXP 或 NOT REGEXP 运算符 (或 RLIKE 和 NOT RLIKE) 来操作正则表达式
 ```SQL
/*选取name以G、F、s开头记录*/
SELECT * FROM Websites WHERE name REGEXP '^[GFs]';
```

### IN操作符(枚举匹配)
- IN 操作符允许您在 WHERE 子句中规定多个值。
```SQL
/*column_name必须是IN规避的值其中之一*/
SELECT column_name(s) FROM table_name WHERE column_name IN (value1,value2,...);

/*name必须是Google，BaiDu其中之一*/
SELECT * FROM Websites WHERE name IN ('Google','BaiDu');
```

### BETWEEN操作符(选取区间值)
- BETWEEN 操作符用于选取介于两个值之间的数据范围内的值。
```SQL

SELECT column_name(s) FROM table_name WHERE column_name BETWEEN value1 AND value2;

/*选取alexa >=1&&<=20的记录*/
SELECT * FROM Websites WHERE alexa BETWEEN 1 AND 20;

/*选取alexa 不是>=1&&<=20的记录*/
SELECT * FROM Websites WHERE alexa NOT BETWEEN 1 AND 20;

/*alexa>=1&&<=20 且country不为'USA','IND'*/
SELECT * FROM Websites WHERE (alexa BETWEEN 1 AND 20) AND country NOT IN ('USA', 'IND');

/*选取 name 以介于 'A' 和 'H' 之间字母开始的所有网站：*/
SELECT * FROM Websites WHERE name BETWEEN 'A' AND 'H';


/*选取 date 介于 '2016-05-10' 和 '2016-05-14' 之间的所有访问记录*/
SELECT * FROM access_log WHERE date BETWEEN '2016-05-10' AND '2016-05-14';
```

### 别名
- 通过使用 SQL，可以为表名称或列名称指定别名。
```SQL

/*列别名*/
SELECT column_name AS alias_name FROM table_name;

/*name、country输出到结果集时分别为n c*/
SELECT name AS n, country AS c FROM Websites;

/*表别名*/
SELECT column_name(s) FROM table_name AS alias_name;

/*表Websites别名为w，可直接使用*/
SELECT w.name, w.url, a.count, a.date FROM Websites AS w, access_log AS a WHERE a.site_id=w.id and w.name="测试";

/*合并url,alexa,country并重命名为site_info*/
SELECT name, CONCAT(url, ', ', alexa, ', ', country) AS site_info FROM Websites;
```
- 使用场景
    - 在查询中涉及超过一个表
    - 在查询中使用了函数
    - 列名称很长或者可读性差
    - 需要把两个列或者多个列结合在一起
 
 ### JOIN(关联)
 - JOIN主要用于关联查询，一般进行关联查询的表都有共同字段
     - 基于共同字段，将多个表合并成一个临时表再查询
     - 一般使用`ON`来确定共同字段
 - 语法
     - `table1 JOIN table2 ON table1.field=table2.field2`
 - 不同的关联
     - `INNER JOIN`
         - 内关联
         - 返回两个表的交集
     - `LEFT JOIN`
         - 以左表为基准表，添加右表中共同字段对应的记录
         - 返回结果集为：左表所有记录+右表中有左表共同字段的记录
     - `RIGHT JOIN`
         - 以右表为基准表，添加左表中共同字段对应的记录
         - 返回结果集为：右表所有记录+左表中有右表共同字段的记录

![](sql.png)

### UNION(合并多个结果集)
-  UNION 操作符合并两个或多个 SELECT 语句的结果。
-  默认地，UNION 操作符选取不同的值。如果允许重复的值，请使用 UNION ALL

```SQL
SELECT column_name(s) FROM table1 UNION SELECT column_name(s) FROM table2;


SELECT column_name(s) FROM table1 UNION ALL SELECT column_name(s) FROM table2;
```

### SELECT INTO、INSERT INTO ... SELECT(复制表)
```SQL
/*复制所有的列插入到新表中*/
SELECT * INTO newtable [IN externaldb] FROM table1;

/*只复制希望的列插入到新表中*/
SELECT column_name(s) INTO newtable [IN externaldb] FROM table1;


/*MySql支持的复制表*/
INSERT INTO table2 SELECT * FROM table1;

/*拷贝特定列*/
INSERT INTO table2(column_name(s)) SELECT column_name(s) FROM table1;

/*拷贝表结构及数据也可以用下面的*/
CREATE TABLE 新表
AS
SELECT * FROM 旧表 
```

### CREATE DATABASE(创建数据库)
```SQL
CREATE DATABASE dbname;

/*创建名为my_db的数据库*/
CREATE DATABASE my_db;
```

### CREATE TABLE(创建表)
```SQL
CREATE TABLE table_name (
column_name1 data_type (size),
column_name2 data_type (size),
column_name3 data_type (size)
,....);


CREATE TABLE Persons (
PersonID int,
LastName varchar(255),
FirstName varchar(255),
Address varchar(255),
City varchar(255)
);
```

### 约束(规定数据的规则)
- SQL 约束用于规定表中的数据规则。如果存在违反约束的数据行为，行为会被约束终止。约束可以在创建表时规定（通过 CREATE TABLE 语句），或者在表创建之后规定（通过 ALTER TABLE 语句）
```SQL

CREATE TABLE table_name (
column_name1 data_type (size) constraint_name,
column_name2 data_type (size) constraint_name,
column_name3 data_type (size) constraint_name
,....);
```
- 常用约束
    - `NOT NULL`
        - 指示某列不能存储 NULL 值。
    - `UNIQUE `
        - 保证某列的每行必须有唯一的值
    - `PRIMARY KEY`主键
        - NOT NULL 和 UNIQUE 的结合。确保某列（或两个列多个列的结合）有唯一标识，有助于更容易更快速地找到表中的一个特定的记录
    - `FOREIGN KEY`外键
        - 保证一个表中的数据匹配另一个表中的值的参照完整性
    - `CHECK`
        - CHECK 约束用于限制列中的值的范围
    - `DEFAULT `
        - 规定列的默认值
```SQL
CREATE TABLE Persons
(
P_Id int NOT NULL, /* P_Id 不能为空*/
LastName varchar(255) NOT NULL,
FirstName varchar(255),
Address varchar(255),
City varchar(255)
)


CREATE TABLE Persons
(P_Id int NOT NULL,
LastName varchar(255) NOT NULL,
FirstName varchar(255),
Address varchar(255),
City varchar(255),
UNIQUE (P_Id)) /*P_Id要保证唯一性*/


CREATE TABLE Persons(
P_Id int NOT NULL,
LastName varchar(255) NOT NULL,
FirstName varchar(255),
Address varchar(255),
City varchar(255),
PRIMARY KEY (P_Id))/*设置主键*/



CREATE TABLE Persons(
P_Id int NOT NULL,
LastName varchar(255) NOT NULL,
FirstName varchar(255),
Address varchar(255),
City varchar(255),
CHECK (P_Id>0)) /*P_Id必须>0*/


CREATE TABLE Persons(
    P_Id int NOT NULL,
    LastName varchar(255) NOT NULL,
    FirstName varchar(255),
    Address varchar(255),
    City varchar(255) DEFAULT 'HeF') /*City列未设置值时默认值为HeF*/

/*创建后添加约束*/
ALTER TABLE Persons ADD UNIQUE (P_Id)

/*创建后删除约束*/
ALTER TABLE Persons DROP INDEX uc_PersonID
```

### INDEX(加快检索速度)
- `CREATE INDEX` 语句用于在表中创建索引。在不读取整个表的情况下，索引使数据库应用程序可以更快地查找数据
- 用户无法看到索引，它们只能被用来加速搜索/查询。
- 更新一个包含索引的表需要比更新一个没有索引的表花费更多的时间，这是由于索引本身也需要更新。因此，理想的做法是仅仅在常常被搜索的列（以及表）上面创建索引
```SQL
/*创建索引*/
CREATE INDEX index_name ON table_name (column_name)

/*创建唯一索引*/
CREATE UNIQUE INDEX index_name ON table_name (column_name)

/*在 "Persons" 表的 "LastName" 列上创建一个名为 "PIndex" 的索引*/
CREATE INDEX PIndex ON Persons (LastName)

/*基于LastName、FirstName创建索引PIndex*/
CREATE INDEX PIndex ON Persons (LastName, FirstName)
```

### DROP(常用于删除操作)
```SQL
/*删除索引*/
DROP INDEX index_name ON table_name

/*删除表*/
DROP TABLE table_name

/*删除数据库*/
DROP DATABASE database_name

/*仅仅删除表内的数据，但并不删除表本身*/
TRUNCATE TABLE table_name
```

### ALTER(修改列)
- ALTER TABLE 语句用于在已有的表中添加、删除或修改列
```SQL
/*添加列*/
ALTER TABLE table_name ADD column_name datatype

/*删除列*/
ALTER TABLE table_name DROP COLUMN column_name

/*修改列的datatype*/
ALTER TABLE table_name MODIFY COLUMN column_name datatype
```

### AUTO INCREMENT (自增)
- Auto-increment 会在新记录插入表中时生成一个唯一的数字
- 常用在生成一个唯一id时使用
```SQL

CREATE TABLE Persons(
ID int NOT NULL AUTO_INCREMENT ,/*id不为空并自增，开始值为1*/
LastName varchar(255) NOT NULL,
FirstName varchar(255),
Address varchar(255),
City varchar(255),
PRIMARY KEY (ID)
)
```

### 视图(结果集的子集)
- 有时，你并不希望客户端通过返回的数据结构，推测出数据库表的结构。这时，可以返回给客户端一个结果集的子集，只给客户端提供它需要的字段，视图就是干这个的；类比过滤器
- 视图包含行和列，就像一个真实的表。视图中的字段就是来自一个或多个数据库中的真实的表中的字段。您可以向视图添加 SQL 函数、WHERE 以及 JOIN 语句，也可以呈现数据，就像这些数据来自于某个单一的表一样
- 视图总是显示最新的数据！每当用户查询视图时，数据库引擎通过使用视图的 SQL 语句重建数据。
- 视图隐藏了底层的表结构，简化了数据访问操作，客户端不再需要知道底层表的结构及其之间的关系。
- 视图提供了一个统一访问数据的接口。（即可以允许用户通过视图访问数据的安全机制，而不授予用户直接访问底层表的权限） 从而加强了安全性，使用户只能看到视图所显示的数据。
- 视图还可以被嵌套，一个视图中可以嵌套另一个视图。
```SQL
/*创建语法*/
CREATE VIEW view_name AS SELECT column_name(s) FROM table_name WHERE condition

/*视图 "Current Product List" 会从 "Products" 表列出所有正在使用的产品（未停产的产品）*/
CREATE VIEW [Current Product List] AS SELECT ProductID,ProductName FROM Products WHERE  Discontinued=No


/*我们可以像这样查询上面这个视图：*/
SELECT * FROM [Current Product List]

/*更新视图，为视图添加Category列*/
CREATE VIEW [Current Product List] AS SELECT ProductID,ProductName,Category FROM Products WHERE Discontinued=No

/*删除视图*/

DROP VIEW view_name
```

### SQL日期函数
| 函数          | 描述                                |
| ------------- | ----------------------------------- |
| NOW()         | 返回当前的日期和时间                |
| CURDATE()     | 返回当前的日期                      |
| CURTIME()     | 返回当前的时间                      |
| DATE()        | 提取日期或日期/时间表达式的日期部分 |
| EXTRACT()     | 返回日期/时间的单独部分             |
| DATE_ADD()    | 向日期添加指定的时间间隔            |
| DATE_SUB()    | 从日期减去指定的时间间隔            |
| DATEDIFF()    | 返回两个日期之间的天数              |
| DATE_FORMAT() | 用不同的格式显示日期/时间           |

### NULL
- 如果表中的某个列是可选的，那么我们可以在不向该列添加值的情况下插入新记录或更新已有的记录。这意味着该字段将以 NULL 值保存
- NULL 用作未知的或不适用的值的占位符
- 无法比较 NULL 和 0；它们是不等价的
- 无法使用比较运算符来测试 NULL 值，比如 =、< 或 <>
- 在MySql中可以使用`IFNULL或COALESCE`在计算时，给NULL赋予特定的值，避免计算出错
    - 必须使用`IS NULL、IS NOT NULL`
```SQL
/*找出Address为NULL的记录*/
SELECT LastName,FirstName,Address FROM Persons WHERE Address IS NULL

/*找出Address不为NULL的记录*/
SELECT LastName,FirstName,Address FROM Persons WHERE Address IS NOT NULL


/*如果alexa列为null值，则赋予0，否则，取原值*/
SELECT id,name,url,IFNULL(alexa,0) FROM websites;
SELECT id,name,url,COALESCE(alexa,0) FROM websites;
```

### 数据类型
- SQL 开发人员必须在创建 SQL 表时决定表中的每个列将要存储的数据的类型。数据类型是一个标签，是便于 SQL 了解每个列期望存储什么类型的数据的指南，它也标识了 SQL 如何与存储的数据进行交互


| 数据类型                           | 描述                                                                                                                 |
| ---------------------------------- | -------------------------------------------------------------------------------------------------------------------- |
| CHARACTER(n)                       | 字符/字符串。固定长度 n。                                                                                            |
| VARCHAR(n) 或 CHARACTER VARYING(n) | 字符/字符串。可变长度。最大长度 n。                                                                                  |
| BINARY(n)                          | 二进制串。固定长度 n。                                                                                               |
| BOOLEAN                            | 存储 TRUE 或 FALSE 值                                                                                                |
| VARBINARY(n) 或BINARY VARYING(n)   | 二进制串。可变长度。最大长度 n。                                                                                     |
| INTEGER(p)                         | 整数值（没有小数点）。精度 p。                                                                                       |
| SMALLINT                           | 整数值（没有小数点）。精度 5。                                                                                       |
| INTEGER                            | 整数值（没有小数点）。精度 10。                                                                                      |
| BIGINT                             | 整数值（没有小数点）。精度 19。                                                                                      |
| DECIMAL(p,s)                       | 精确数值，精度 p，小数点后位数 s。例如：decimal(5,2) 是一个小数点前有 3 位数，小数点后有 2 位数的数字。              |
| NUMERIC(p,s)                       | 精确数值，精度 p，小数点后位数 s。（与 DECIMAL 相同）                                                                |
| FLOAT(p)                           | 近似数值，尾数精度 p。一个采用以 10 为基数的指数计数法的浮点数。该类型的 size 参数由一个指定最小精度的单一数字组成。 |
| REAL                               | 近似数值，尾数精度 7。                                                                                               |
| FLOAT                              | 近似数值，尾数精度 16。                                                                                              |
| DOUBLE PRECISION                   | 近似数值，尾数精度 16。                                                                                              |
| DATE                               | 存储年、月、日的值。                                                                                                 |
| TIME                               | 存储小时、分、秒的值。                                                                                               |
| TIMESTAMP                          | 存储年、月、日、小时、分、秒的值。                                                                                   |
| INTERVAL                           | 由一些整数字段组成，代表一段时间，取决于区间的类型。                                                                 |
| ARRAY                              | 元素的固定长度的有序集合                                                                                             |
| MULTISET                           | 元素的可变长度的无序集合                                                                                             |
| XML                                | 存储 XML 数据                                                                                                        |

### MySql数据类型
- 在 MySQL 中，有三种主要的类型：Text（文本）、Number（数字）和 Date/Time（日期/时间）类型
- Text 类型

| 数据类型         | 描述                                                                                                                                                                                       |
| ---------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| CHAR(size)       | 保存固定长度的字符串（可包含字母、数字以及特殊字符）。在括号中指定字符串的长度。最多 255 个字符。                                                                                          |
| VARCHAR(size)    | 保存可变长度的字符串（可包含字母、数字以及特殊字符）。在括号中指定字符串的最大长度。最多 255 个字符。注释：如果值的长度大于 255，则被转换为 TEXT 类型。                                    |
| TINYTEXT         | 存放最大长度为 255 个字符的字符串。                                                                                                                                                        |
| TEXT             | 存放最大长度为 65,535 个字符的字符串。                                                                                                                                                     |
| BLOB             | 用于 BLOBs（Binary Large OBjects）。存放最多 65,535 字节的数据。                                                                                                                           |
| MEDIUMTEXT       | 存放最大长度为 16,777,215 个字符的字符串。                                                                                                                                                 |
| MEDIUMBLOB       | 用于 BLOBs（Binary Large OBjects）。存放最多 16,777,215 字节的数据。                                                                                                                       |
| LONGTEXT         | 存放最大长度为 4,294,967,295 个字符的字符串。                                                                                                                                              |
| LONGBLOB         | 用于 BLOBs (Binary Large OBjects)。存放最多 4,294,967,295 字节的数据。                                                                                                                     |
| ENUM(x,y,z,etc.) | 允许您输入可能值的列表。可以在 ENUM 列表中列出最大 65535 个值。如果列表中不存在插入的值，则插入空值。 注释：这些值是按照您输入的顺序排序的。可以按照此格式输入可能的值： ENUM('X','Y','Z') |
| SET              | 与 ENUM 类似，不同的是，SET 最多只能包含 64 个列表项且 SET 可存储一个以上的选择。                                                                                                          |


- Number类型
    - 下面的size代表的并不是存储在数据库中的具体的长度，如 int(4) 并不是只能存储4个长度的数字。实际上int(size)所占多少存储空间并无任何关系。int(3)、int(4)、int(8) 在磁盘上都是占用 4 btyes 的存储空间。就是在显示给用户的方式有点不同外，int(M) 跟 int 数据类型是相同的。就是显示的长度不一样而已 都是占用四个字节的空间
```SQL
int（9）显示结果为000000010
int（3）显示结果为010
```

| 数据类型        | 描述                                                                                                                  |
| --------------- | --------------------------------------------------------------------------------------------------------------------- |
| TINYINT(size)   | 带符号-128到127 ，无符号0到255。                                                                                      |
| SMALLINT(size)  | 带符号范围-32768到32767，无符号0到65535, size 默认为 6。                                                              |
| MEDIUMINT(size) | 带符号范围-8388608到8388607，无符号的范围是0到16777215。 size 默认为9                                                 |
| INT(size)       | 带符号范围-2147483648到2147483647，无符号的范围是0到4294967295。 size 默认为 11                                       |
| BIGINT(size)    | 带符号的范围是-9223372036854775808到9223372036854775807，无符号的范围是0到18446744073709551615。size 默认为 20        |
| FLOAT(size,d)   | 带有浮动小数点的小数字。在 size 参数中规定显示最大位数。在 d 参数中规定小数点右侧的最大位数。                         |
| DOUBLE(size,d)  | 带有浮动小数点的大数字。在 size 参数中规显示定最大位数。在 d 参数中规定小数点右侧的最大位数。                         |
| DECIMAL(size,d) | 作为字符串存储的 DOUBLE 类型，允许固定的小数点。在 size 参数中规定显示最大位数。在 d 参数中规定小数点右侧的最大位数。 |

- Date 类型

| 数据类型    | 描述                                                                                                                                                                                  |
| ----------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| DATE()      | 日期。格式：YYYY-MM-DD 注释：支持的范围是从 '1000-01-01' 到 '9999-12-31'                                                                                                              |
| DATETIME()  | 日期和时间的组合。格式：YYYY-MM-DD HH:MM:SS 注释：支持的范围是从 '1000-01-01 00:00:00' 到 '9999-12-31 23:59:59'                                                                       |
| TIMESTAMP() | 时间戳。TIMESTAMP 值使用 Unix 纪元('1970-01-01 00:00:00' UTC) 至今的秒数来存储。格式：YYYY-MM-DD HH:MM:SS 注释：支持的范围是从 '1970-01-01 00:00:01' UTC 到 '2038-01-09 03:14:07' UTC |
| TIME()      | 时间。格式：HH:MM:SS 注释：支持的范围是从 '-838:59:59' 到 '838:59:59'                                                                                                                 |
| YEAR()      | 2 位或 4 位格式的年。 注释：4 位格式所允许的值：1901 到 2155。2 位格式所允许的值：70 到 69，表示从 1970 到 2069。                                                                     |
- 即便 DATETIME 和 TIMESTAMP 返回相同的格式，它们的工作方式很不同。在 INSERT 或 UPDATE 查询中，TIMESTAMP 自动把自身设置为当前的日期和时间。TIMESTAMP 也接受不同的格式，比如 YYYYMMDDHHMMSS、YYMMDDHHMMSS、YYYYMMDD 或 YYMMDD


### SQL函数
- SQL内置一些函数
- SQL Aggregate 函数
    - SQL Aggregate 函数计算从列中取得的值，返回一个单一的值  
        - AVG() - 返回平均值
        - COUNT() - 返回返回匹配指定条件的行数,NULL 不计入
        - FIRST() - 返回第一个记录的值
        - LAST() - 返回最后一个记录的值
        - MAX() - 返回最大值
        - MIN() - 返回最小值
        - SUM() - 返回总和
- SQL Scalar 函数
    - SQL Scalar 函数基于输入值，返回一个单一的值
        - UCASE() - 将某个字段转换为大写
        - LCASE() - 将某个字段转换为小写
        - MID() - 从某个文本字段提取字符，MySql 中使用SubString(字段，1，end) - 从某个文本字段提取字符
        - LEN() - 返回某个文本字段的长度
        - ROUND() - 对某个数值字段进行指定小数位数的四舍五入
        - NOW() - 返回当前的系统日期和时间
        - FORMAT() - 格式化某个字段的显示方式
```SQL
/*从 "access_log" 表的 "count" 列获取平均值*/
SELECT AVG(count) AS CountAverage FROM access_log;

/* "access_log" 表中 "site_id"=3 的总访问量，NULL 不计入*/
SELECT COUNT(count) AS nums FROM access_log WHERE site_id=3;

/* "access_log" 表中总记录数*/
SELECT COUNT(*) AS nums FROM access_log;

/* "access_log" 表中不同 site_id 的记录数*/
SELECT COUNT(DISTINCT site_id) AS nums FROM access_log;

/*First()只有 MS Access支持。MySql中可通过LIMIT返回首条记录*/

/*选取 "Websites" 表的 "name" 列中第一个记录的值*/
SELECT name AS FirstSite FROM Websites LIMIT 1;

/*只有 MS Access 支持 LAST() 函数，MySql中可通过降序+LIMIT拿到最后一个记录*/
SELECT name FROM WebsitesORDER BY id DESC LIMIT 1;

/*从 "Websites" 表的 "alexa" 列获取最大值*/
SELECT MAX(alexa) AS max_alexa FROM Websites;

/*从 "Websites" 表的 "alexa" 列获取最小值*/
SELECT MIN(alexa) AS min_alexa FROM Websites;

/*查找 "access_log" 表的 "count" 字段的总数*/
SELECT SUM(count) AS nums FROM access_log;

/*从 "Websites" 表中选取 "name" 和 "url" 列，并把 "name" 列的值转换为大写*/
SELECT UCASE(name) AS site_title, url FROM Websites;

/*从 "Websites" 表中选取 "name" 和 "url" 列，并把 "name" 列的值转换为小写*/
SELECT LCASE(name) AS site_title, urlFROM Websites;

/*从 "Websites" 表的 "name" 列中提取前 4 个字符*/
SELECT MID(name,1,4) AS ShortTitleFROM Websites;

/*从 "Websites" 表中选取 "name" 和 "url" 列中值的长度*/
SELECT name, LENGTH(url) as LengthOfURLFROM Websites;

/*假设price为1.32*/
SELECT ROUND(price, 1) FROM base_price /*1.3*/
SELECT ROUND(price, 0) FROM base_price/*1*/
SELECT ROUND(price) FROM base_price /*1.3*/
        
/*从 "Websites" 表中选取 name，url，及当天日期*/
SELECT name, url, Now() AS dateFROM Websites;

/*从 "Websites" 表中选取 name, url 以及格式化为 YYYY-MM-DD 的日期*/
SELECT name, url, DATE_FORMAT(Now(),'%Y-%m-%d') AS dateFROM Websites;
```

### GROUP BY、HAVING(聚合、分组)
- GROUP BY 语句用于结合聚合函数，根据一个或多个列对结果集进行分组
```SQL
/*将SELECT结果集根据site_id分组聚合*/
SELECT site_id, SUM(access_log.count) AS nums FROM access_log GROUP BY site_id;

/*关联多个表进行分组*/
SELECT Websites.name,COUNT(access_log.aid) AS nums FROM access_log LEFT JOIN WebsitesON access_log.site_id=Websites.id GROUP BY Websites.name;

/*分组后筛选出count>200的记录*/
SELECT Websites.name, Websites.url, SUM(access_log.count) AS nums FROM (access_logINNER JOIN Websites ON access_log.site_id=Websites.id) GROUP BY Websites.name HAVING SUM(access_log.count) > 200;
```

## 参考
> https://www.runoob.com/sql/sql-tutorial.html
