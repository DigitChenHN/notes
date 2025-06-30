# SQL 
**S**tructured **Q**uery **L**anguage，结构化查询语言，是用来访问和处理关系型数据库的编程语言。  
对于理解关系型数据库，可以想象二维表格，二维表中每一行为一个记录，每一列为一个字段。不同表格存在关系的话，就通过**外键**关联。  
**SQL语句不区分大小写**，但是建议关键字大写，表名、列名小写。  
> 本文主要是对b站视频[一小时MySQL教程](https://www.bilibili.com/video/BV1AX4y147tA/?spm_id_from=333.337.search-card.all.click&vd_source=d82de55cfde970cdf86016bef2c6de4e)笔记，主要从一个初学者的角度记录对关系型数据库的理解和SQL语言特性，至于具体的语法，视频对应的公众号会提供相应的总结笔记，这里可能很多不会记录；并且由于本文不展示语句执行的结果，所以理解起来应该是十分抽象的。  

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
`table_name`后面的列名如果与表格中的列数量、顺序一致的话，则可以省略；另外，在VALUES后面可以加入多个括号以插入多条数据。  
- 查找记录的关键字`SELECT`：
```SQL
SELECT * FROM table_name;
```
`*`处可以为列名（`*`为通配符，代表所有列）。

- 更新更改表格的数据记录的关键字`UPDATE`：
```SQL
UPDATE table_name SET column1=value1, column2=value2, ... WHERE condition;
```
- 删除记录的关键字`DELETE`:
```SQL
DELETE FROM table_name WHERE condition;
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
WHERE子句用来指定某些满足标准的记录。经常跟`SELECT`，`UPDATE`，`DELETE`等语句一起使用。  
WHERE后面跟的是一个条件，经常使用的是`>`，`=`，`<`，`>=`，`<=`，`!=`，`BETWEEN...AND...`，`IN`，`LIKE`，`IS NULL`，`AND`，`OR`，`NOT`等组成逻辑判断语句。  
其中特别注意的有：  
- `IN`，`SELECT * FROM table_name WHERE column_name IN (value1, value2, ...)`
- `LIKE`关键词，用来表示模糊匹配，后面的匹配模式中，`%`表示任意多个字符，`_`表示任意一个字符。跟正则表达式类似。  
- 如果匹配模式比较复杂，则需要使用`REGEXP`关键词。`SELECT * FROM table_name WHERE column_name REGEXP 'pattern'`  
- 如果特别需要匹配空值，则使用`IS NULL`关键词。而不能用`=`。

### 排序ORDER BY  
ORDER BY用来对结果集进行排序，默认是升序排列`ASC`，降序排列则在列名的后面加`DESC`。  
```SQL
SELECT * FROM table_name ORDER BY column_name DESC;
```
> 列名也可以用序号来代替。  
### 计算平均值、计数等——聚合函数  
理解聚合函数：聚合函数可以理解为对列操作，常见的聚合函数有`COUNT`、`AVG`、`SUM`、`MAX`、`MIN`，函数的参数传入列名就代表对该列进行操作，比如`SELECT AVG(column_name) FROM table_name`会返回一个值，就是列的平均值。tips：可以将聚合函数想象成增加一列新的数据，数据的结果是对列的操作结果。  
- `COUNT`：计算行数，`SELECT COUNT(*) FROM table_name`
- `GROUP BY`：分组；
  - 理解`GROUP BY`：比如`SELECT COUNT(*) FROM table_name`会将所有行无差别进行计数，但`SELECT COUNT(*) FROM table_name GROUP BY gender`，则会将所有记录先按照gender（性别）这一列分类，然后对每一个实现COUNT，也就是说这个语句会返回两行值，分别是gender中为“女”的计数结果和为“男”的计数结果。如果使用`SELECT gender, COUNT(*) FROM table_name GROUP BY gender`，则会返回两列，分别是gender和COUNT(*)，即性别和计数结果。
  - 常常和`having`关键字一起使用，`SELECT gender, COUNT(*) FROM table_anme GROUP BY gender HAVING COUNT(*) > 4`，`having`后面跟的是条件，用来筛选分组之后的数据。在上面这个例子中，gender中数量大于4的才会被返回。  
- `LIMIT`限制显示的行数。   
```SQL
SELECT SUBSTR(name, 1, 1), COUNT((SUBSTR(name, 1, 1))) FROM table_name -- 选择name列的第一个字符（姓氏），然后对该字符进行计数
GROUP BY SUBSTR(name, 1, 1) -- 按照姓氏进行分组
HAVING COUNT((SUBSTR(name, 1, 1))) > 4 -- 筛选出姓氏数量大于4的
ORDER BY COUNT((SUBSTR(name, 1, 1))) DESC -- 按照数量降序排列
LIMIT 2; -- 只显示前两行
```
以上命令整体结果就是将人数排名前两名的姓氏显示出来。  
```SQL
LIMIT 3, 3; -- 从第4行开始显示3行，代入到上面的例子中，就是显示人数排名第4、5、6的姓氏。
```
以上过程也是“分页查询”的原理，:rofl:虽然我也不知道什么是分页查询。

- `DISTINCT`用于去重。
```SQL
SELECT DISTINCT column_name FROM table_name;
```
以上语句会返回一列，列中的值是去重之后的column_name列的值。 

- `UNION`用于合并两个结果集。将上述的语句用UNION连接起来，就可以得到两个结果集的**并集**。  
```SQL
SELECT * FROM table_name WHERE column1 BETWEEN 20 AND 30
UNION
SELECT * FROM table_name WHERE column2 BETWEEN 40 AND 50;
```
`UNION`默认会将两个结果中重复的记录进行去重（只保留一条），如果需要保留重复的记录，则使用`UNION ALL`。

- 相对于`UNION`得到结果的并集，`INTERSECT`得到的是两个结果的交集。
- `EXCEPT`得到的是两个结果的差集。

## 子查询  
查询的关键字为`SELECT`。`SELECT`后面跟的内容其实就是想要的结果中的各个列。  
如果想将一个查询的结果作为另一个查询的条件，则可以使用子查询。
```SQL
SELECT level, 
ROUND(SELECT(AVG(level) FROM table_name)) as average,
level - ROUND(SELECT(AVG(level) FROM table_name)) as diff 
FROM table_name;  
```
1. 以上语句中使用到了子查询。
2. <div id='anchor1'>SELECT后面跟的是结果想要的列，即结果中第一列应该是原表格的level，第二列是原表格level列的平均数，第三列则是原表格level的值与平均数的差。</div>
3. 其中使用了as，为新列进行命名。  

此外，子查询也可以用在`CREATE`、`INSERT INTO`、`UPDATE`、`DELETE`等语句中。
```SQL
CREATE TABLE new_table SELECT * FROM table_name WHERE column_name > (SELECT AVG(column_name) FROM table_name);
```
以上语句中，创建了一个新的表格new_table，其中的内容是原表格table_name中column_name大于原表格column_name平均值的记录。  

### exists关键字
exists关键字用于判断一个子查询是否有结果，其返回的内容只有0和1。
```SQL
SELECT exists(SELECT * FROM table_name WHERE level > 10);
```
## 表关联
1. 表关联用来查询多个表中的数据。
2. 关联的表中必须有相同的字段。  
3. 一般会使用表的主键和外键来关联。（键应该就是列名的意思，主键和外键应该是某些特殊的列名）
4. 主要有几种种连接类型：
   1. 内连接`INNER JOIN`，返回两个表中都存在的记录。
   2. 左连接`LEFT JOIN`，返回左表中所有的记录和右表中匹配的记录。
   3. 右连接`RIGHT JOIN`，返回右表中所有的记录和左表中匹配的记录。

### 用法示例  
首先有一个表player来存储玩家信息，其中包括的字段有“id”、“name”、“level”、“gender”、“age”、“country”。接着使用另一个表equip来存储玩家的装备信息，其中包括的字段有“id”、“player_id”、“equip_name”。
往往数据表会有一个“id”字段作为主键，主键是用来唯一表示一条记录的，如果很难理解可以想象pandas中的index或者excel中的行号。所以在player表中，同样也有一个名为“id”的主键，代表每一条记录。
而外键是用来表示两个表之间的关系的，在这个例子中，要表示“哪一件装备是属于哪一个玩家的”，需要将两个表的对应记录连接起来，这时候就需要用到外键。可以考虑将equip表中的“player_id”作为外键，用来连接player表中的”id“。使用`INNER JOIN`：
```SQL
SELECT * FROM player
INNER JOIN equip
on player.id = equip.player_id;  -- 匹配条件
```
这个将会返回一个新的表格，将所有equip中的列添加到player表格后面成为一个新的表格，并且会将equip中的player_id和player中的id进行匹配，并且由于使用了`INNER JOIN`，所以只会返回两个表中都存在的记录，即player_id和player中id能够匹配的上的记录。具体到例子中就是只有有装备的玩家才会出现在新表格中。   
相应的，如果使用`LEFT JOIN`：
```SQL
SELECT * FROM player
LEFT JOIN equip
on player.id = equip.player_id;
```
`LEFT JOIN`将保留左表中的所有记录，然后匹配右表中的数据。这样一来，结果表中将包含player表中所有的记录，并且会拼接右表中的所有列，但是player.id匹配不到equip.player_id的记录，将使用NULL填充。  
`RIGHT JOIN`就很容易理解了。  

表连接的效果也可以直接通过`WHERE`语句实现。  
```SQL
SELECT * FROM player, equip WHERE player.id = equip.player_id
```
[前面](#anchor1)提到了`SELECT`后面接的是结果表的列，所以上面的这个语句实际上就是将两个表格左右拼接起来（按照条件匹配）。  
考虑一种情况，如果没有WHERE语句，结果表格应该是怎么样的呢？  
可能有些人会觉得是两个表直接拼接在一起，即：  
|id|name|level|gender|age|country|id|player_id|equip_name|  
|--|--|--|--|--|--|--|--|--|  
|1|li|3|女|14|中国|1|3|锤子|  
|2|wang|34|女|14|中国|2|3|锤子|  
|3|chen|5|男|14|中国|NULL|NULL|NULL|  

但实际上这种方式就是按照id进行匹配，等价于：
```SQL
SELECT * FROM player
LEFT JOIN equip
on player.id = equip.id
```
当你不提供条件的时候，sql是不知道按照哪一列进行匹配的，所以会返回的是两个表的笛卡尔积，即player表格中的每一条记录都和equip中所有记录进行匹配然后拼接，也就是说，结果表格中的记录条数=player记录条数$\times$equip记录条数。  
从以上分析我们可以知道，在两个表的笛卡尔积的基础上增加条件判断语句进行筛选，就可以得到匹配好的结果了。  
```SQL
SELECT * FROM player, equip WHERE player.id = equip.player_id
```

## 索引  
普通的查询语句：`SELECT * FROM table_name WHERE level > 4`，将遍历所有的数据来找到符合条件的记录，这样的查询方式在数据量非常多的情况下可能效率较低。  

创建索引：
```SQL
CREATE [UNIQUE|FULLTEXT|SPATIAL] INDEX index_name
ON table_name (index_col_name,...);
```
一般为WHERE语句后面的条件列创建索引：
```SQL
CREATE INDEX email_index ON fast ( email); 
```
其中“email”为列名，“email_index”为索引名，fast为表名，创建如上索引之后，执行同样的查询语句的时间将会有很大区别。  
删除索引：
```SQL
DROP INDEX email_index ON fast;
```
查看表中存在的索引： 
```SQL
SHOW INDEX FORM fast;
```

## 视图
视图是一种虚拟存在的表。本身不存放任何数据，而是存放查询语句。  
创建视图的方式：  
```SQL
CREATE VIEW top10
AS 
SELECT * FROM player ORDER BY level DESC LIMIT 10;
```
视图的创建方式：CREATE VIEW + 视图名 接 AS + 查询语句。  
以上方式创建了一个视图top10，用来查询表格player中level最高的10条记录。  
查看视图的结果需要将其视为一个表格，使用SELECT语句：
```SQL
SELECT * FROM top10;
```
由于视图中存放的是查询语句，所以能够动态更新查询的结果，即一旦原表格发生改变，视图展示的结果也会发生改变。  

修改视图（修改表格的属性和结构用`ALTER TABLE`，修改表格的内容用`UPDATE`，因此修改视图的结构用`ALTER VIEW`）：   
```SQL
ALTER VIEW top10 
AS
SELECT * FROM player ORDER BY level LIMIT 10;
```

删除视图：  
```SQL
DROP VIEW top10;
``` 