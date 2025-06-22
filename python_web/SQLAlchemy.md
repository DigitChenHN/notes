## 简介
SQLAlchemy是一个ORM（object relation mapping）框架，它提供了高层的API，可以通过对象的方式操作数据库。flask-sqlalchemy库为flask提供了SQLAlchemy插件使我们可以方便的将sql数据库中的各种数据表映射为python对象，从而可以以操作对象的方式来操作数据库。

## 定义数据库
```python 
# __init__.py

from flask_sqlalchemy import SQLAlchemy   
from flask import Flask
from config import Config

app = Flask(__name__) 

def create_app(config_class=Config):
    app.config.from_object(config_class) # 设定app的配置文件

    db = SQLAlchemy() 
    db.init_app(app)

    return app
```
这里关于Config这个配置类是如何定义的，我们暂时不做了解，只需要知道其中定义了一个SQLALCHEMY_DATABASE_URI配置项，该配置项用于指定数据库的连接信息。  
该配置项的值是一个字符串，格式为：dialect+driver://username:password@host:port/database。  
比如“sqlite:///app.db”，表示使用SQLite数据库，数据库文件名为app.db。这样一来就在项目根目录处生成了一个app.db的数据库文件。除了在项目代码中使用SQLAlchemy之外，也可也使用python的sqlite3模块来操作这个数据库文件，这部分会在[sqlite3](/python_web/sqlite3.md)中介绍。  
其中dialect表示数据库的类型，driver表示数据库的驱动，username表示用户名，password表示密码，host表示主机，port表示端口号，database表示数据库名。  
除了SQLite之外，SQLAlchemy还支持MySQL、PostgreSQL、Oracle、Microsoft SQL Server等多种数据库。  

## 定义数据库模型
在一个web服务中，需要使用数据库来存储用户的各种数据。  
```python 
from datetime import datetime
from app import db

class UserLocation(db.Model):
    """用户位置模型"""
    id = db.Column(db.Integer, primary_key=True)
    user_id = db.Column(db.Integer, db.ForeignKey('user.id'), nullable=False)
    latitude = db.Column(db.Float, nullable=False)
    longitude = db.Column(db.Float, nullable=False)
    city = db.Column(db.String(100))
    last_updated = db.Column(db.DateTime, default=datetime.utcnow)
    
    def __repr__(self):
        return f'<UserLocation {self.city} ({self.latitude}, {self.longitude})>'
```
以上代码片段定义了一个UserLocation类来存放用户的位置信息，这个类的定义好之后就可以理解为给db这个数据库添加了一个名为user_location的表。其中id为主键，user_id为外键，指向User模型中的id字段。关于主键外键的理解可以参考[文章](/mySQL.md)  

> 该类继承自db.Model类，db.Model类是SQLAlchemy中定义的基类，它为数据库表提供了模型。其中的db是一个SQLAlchemy的实例`db = SQLAlchemy()`，在整个应用app的`__init__.py`文件中定义。  
> SQLAlchemy 生成表名的规则如下：  
    1. 默认规则 ：如果未显式指定表名，SQLAlchemy 会根据模型类的名称自动生成表名。默认情况下，表名是模型类名的小写形式，并将驼峰命名转换为下划线分隔。例如，模型类 UserLocation 会生成表名 user_location 。  
    2. 显式指定表名 ：可以通过在模型类中定义 __tablename__ 属性来显式指定表名。例如：
 ```python 
 class UserLocation(db.Model):
     __tablename__ = 'location'
     // ... existing code ...
 ```

## 使用数据库模型   
向数据表添加记录：  
```python 
# 更新或创建用户位置记录
user_location = UserLocation.query.filter_by(user_id=current_user.id).first() or \
            UserLocation(user_id=current_user.id)
user_location.latitude = latitude
user_location.longitude = longitude
user_location.city = location_info.get('city')

db.session.add(user_location)
db.session.commit()
```
db.session表示一个数据库会话，用于执行数据库操作。  
`db.session.add()`方法用于向数据库会话中添加一个对象。
`db.session.commit()`方法用于提交数据库会话，使所有添加、修改、删除的记录生效。提交之后user_location数据将被持久化到数据库中。  

## 读取数据库数据
当web应用需要读取用户数据时，就可以将导入这个类，然后对数据进行操作：
```python 
import UserLocation

user_location = UserLocation.query.filter_by(user_id=self.user_id).first()
```
`query.filter_by()`方法用于查询数据库中符合条件的，first()将会返回查询到的第一个结果。  
整个语句返回的是一条数据库记录（作为一个python的对象），因此可以用对象的属性来访问该条数据的值，比如`user_location.latitude`。  

## 修改数据库数据
当需要修改数据库时，直接修改对象的属性，并且在最后将修改通过`db.session.commit()`提交即可。
```python 
config = UserLLMConfig.query.filter_by(user_id=current_user.id).first()

config.model_type = form.model_type.data
config.api_key = form.api_key.data.strip()

db.session.commit()
```

## 删除数据库记录
```python 
# 删除用户位置记录
user_location = UserLocation.query.filter_by(user_id=current_user.id).first()
db.session.delete(user_location)
db.session.commit()
```