# sqlite3
SQLite是一个轻量级的关系型数据库，适合小型应用和嵌入式系统，它不需要安装和配置，直接使用文件存储数据。sqlite3是python中的一个库，可以用来操作SQLite数据库。  

# 基本使用方法
使用SQLite本质上还是使用SQL语句，所以语法和[MySQL](/mySQL.md)等关系型数据库几乎是一致的。本文主要介绍的是sqlite3这个python库的一些类和方法。  

## 连接数据库
在操作数据库之前，首先需要链接到数据库，而对于sqlite3，就是链接到一个数据库文件，**使用`.connect()`方法**，如果该文件不存在，则会自动创建。链接数据库的代码如下：  
```python
import sqlite3
# 链接到数据库文件，如果文件不存在则会自动创建
conn = sqlite3.connect('example.db')
```
## 创建游标执行SQL语句
建立conn对象之后，还需要创建一个游标对象cursor，使用`.cursor()`方法。游标对象是用来执行SQL语句和获取结果的。代码如下：  
```python
cursor = conn.cursor()
```

之后就可以使用游标对象的`.execute()`方法来执行SQL语句了。比如创建一个表：  
```python
cursor.execute('''CREATE TABLE IF NOT EXISTS users (
    id INTEGER PRIMARY KEY AUTOINCREMENT, 
    name TEXT NOT NULL,
    age INTEGER NOT NULL
)''')
```
执行之后需要通过游标对象的`.fetchall()`或者`.fetchone()`方法来获取结果。  
其中`.fetchall()`方法返回类型为：`list[tuple]`，即所有查询结果的列表，每个元素是一个元组，代表一行查询的数据。而`.fetchone()`则直接返回查询结果的第一行，即直接返回一个元组。  
```python
[
    (col1_val_row1, col2_val_row1, ...),  # 第1行数据
    (col1_val_row2, col2_val_row2, ...),  # 第2行数据
    ...
]
```

如果执行的SQL语句是`INSERT`、`UPDATE`、`DELETE`等修改数据的语句，则需要使用`.commit()`方法来提交事务。代码如下：  
```python
conn.commit()
``` 
