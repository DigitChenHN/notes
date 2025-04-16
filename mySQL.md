# SQL 
**S**tructured **Q**uery **L**anguage，结构化查询语言，是用来访问和处理关系型数据库的编程语言。  
对于理解关系型数据库，可以想象二维表格，二维表中每一行为一个记录，每一列为一个字段。不同表格存在关系的话，就通过**外键**关联。  
**SQL语句不区分大小写**，但是建议关键字大写，表名、列名小写。  
> 本文主要是对b站视频[一小时MySQL教程](https://www.bilibili.com/video/BV1AX4y147tA/?spm_id_from=333.337.search-card.all.click&vd_source=d82de55cfde970cdf86016bef2c6de4e)笔记，主要从一个初学者的角度记录对关系型数据库的理解和SQL语言特性，至于具体的语法，视频对应的公众号会提供相应的总结笔记，这里可能很多不会记录。  

## 建立数据库和数据表
```SQL
CREATE DATABASE database_name;
```
$\uparrow\uparrow\uparrow$建立指定名称的数据库。   
```SQL
USE database_name;
```  
$\uparrow\uparrow\uparrow$使用指定名称的数据库。  

数据库的层级结构为：数据库——表——行——列。  
因此建立一个新的数据库之后，需要通过以下语句建立一个新的表：  
```SQL
CREATE TABLE table_name (
    id INT,
    name VARCHAR(255),
    age INT
)
```
每一个字段需要定义数据的类型，所以需要留意SQL中的数据类型，主要有五大类：***数值类型，日期类型，字符串类型，二进制类型，空间类型***。  
查看表信息：  
```SQL
DESC table_name;
```

## 建立数据表的过程中使用的关键字  
建立数据表一般是对表和列进行操作  
主要的操作关键字有`ALTER`, `ADD`, `DROP`, `MODIFY`, `RENAME`, `TRUNCATE`等。具体含义同样是参看视频和公众号的笔记。  
表和列的关键字：    
- `TABLE`：表。常用如`ALTER TABLE table_name ADD COLUMN column_name INT`  
- `COLUMN`：列。常用如`ADD COLUMN`, `MODIFY COLUMN`（修改数据类型）  
通过以上可以发现，SQL类似于自然语言，主要在于记忆相应的一些操作对应的关键字。  

## 表的增删改查  
- 插入记录的关键字`INSERT`：  
```SQL
INSERT INTO table_name (column1, column2, ...) VALUES (value1, value2, ...);
```  
`table_name`后面的列名如果与表格中的列数量、顺序一致的话，则可以省略。  
- 查找记录的关键字`SELECT`：
```SQL
SELECT * FROM table_name;
```
`*`处可以为列名，`*`为通配符，代表所有列。

- 更新更改表格的数据记录的关键字`UPDATE`：
```SQL
UPDATE table_name SET column1=value1, column2=value2, ... WHERE condition;
```

## 数据的导入导出
导出命令：  
```bash
mysqldump -u root -p database_name table_name > database_name.sql
```
其中的`table_name`可以省略从而导出整个数据库。  
导出的文件本质上是一系列的可以用来重建该数据库的sql语句。  

导入命令：
```bash
mysql -u root -p database_name < database_name.sql
```  

## 常用语句  
***个人认为对于初次接触比较难记住的是各种语句关键词的位置，主要是要记住这些子句关键字针对的对象，比如`SELECT`语句中`FROM`后面跟的是表名，`WHERE`后面跟的是条件，`ORDER BY`后面跟的是列名等。***
### 条件判断——WHERE
WHERE子聚用来指定某些满足标准的记录。经常跟`SELECT`，`UPDATE`，`DELETE`等语句一起使用。  
WHERE后面跟的是一个条件，经常使用的是`>`，`=`，`<`，`>=`，`<=`，`!=`，`BETWEEN...AND...`，`IN`，`LIKE`，`IS NULL`，`AND`，`OR`，`NOT`等组成逻辑判断语句。  
其中特别注意的有：  
- `IN`，`SELECT * FROM table_name WHERE column_name IN (value1, value2, ...)`
- `LIKE`关键词，用来表示模糊匹配，后面的匹配模式中，`%`表示任意多个字符，`_`表示任意一个字符。跟正则表达式类似。  
- 如果匹配模式比较负责，则需要使用`REGEXP`关键词。`SELECT * FROM table_name WHERE column_name REGEXP 'pattern'`  
- 如果特别需要匹配空值，则使用`IS NULL`关键词。而不能用`=`。

### 排序ORDER BY  
ORDER BY用来对结果集进行排序，默认是升序排列`ASC`，降序排列则在列名的后面加`DESC`。  
```SQL
SELECT * FROM table_name ORDER BY column_name DESC;
```
> 列名也可以用序号来代替。  
### 计算平均值、计数等——聚合函数  
- `COUNT`：计算行数，`SELECT COUNT(*) FROM table_name`
- `GROUP BY`：分组；
  - 常常和`having`关键字一起使用，`SELECT column_name, COUNT(*) FROM table_anme GROUP BY column_name HAVING COUNT(*)`，`having`后面跟的是条件，用来筛选分组之后的数据。  
  - 