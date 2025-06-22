# 简单说明
flask是一个基于python的轻量级web框架，其中包含了大量的开发一个web应用常用的功能和扩展，比如一般一个web应用都需要提供给用户注册、登录、数据存储，而flask中就有大量的现成的函数。个人认为真正要理解这些框架的使用，还是需要使用框架去开发一个简单的应用，单纯看文档或者某些系统的教程，很容易陷入大量的术语迷宫中，[菜鸟教程](https://www.runoob.com/flask/flask-tutorial.html)中已经介绍的比较通俗了，但是没有web开发基础，不能结合完整的实际案例的话，仍然容易陷入术语陷阱中。  

# 背景
现在假设我们要开发一个简单的web应用，核心的功能就是为用户提供一个跟大语言模型聊天的界面。  
假设我们的网站域名叫做`chatgpt.example.com`。
- 当用户进入网站的时候，首先展示的是主页面，由于此时是未登录的状态，此时应该展示一个html页面（就命名为index.html吧），页面中有一个登录和一个注册的按钮，这个html作为`主页面`。  
- 当用户点击注册按钮的时候，就要能够跳转到`注册页面`，假设我们希望地址跳转到`chatgpt.example.com/auth/register`，那么这个页面也需要有一个html文件（就命名为register.html吧）来展示注册表单。
    > *注册表单可以让用户填入用户名、密码等信息，然后用户点击注册之后，信息会被提交到服务端，后端的代码会将用户信息存储到数据库中*。  


那么为了实现以上过程：
- 首先需要展示相应的html文件，这些html文件我们可以考虑统统放到项目的`templates`目录下。
- 然后点击某个页面中的按钮，需要与数据库交互之后再跳转到另外一个html页面上，如果是单纯的html页面，那么也许可以考虑给这个元素添加一个`href`属性，并指向另一个html文件（填入该html的路径）。但是在web应用中，是需要更复杂的逻辑的，需要与后端交互之后再展示另一个html页面，所以在flask框架下，`href`属性的值应该是一个**路由**地址。  
- 路由的作用就是将用户的请求映射到相应的处理函数上，在flask中，处理函数就是一个普通的python函数，也叫**视图函数**，通常接收一个请求对象作为参数。比如对于注册和登录，可以考虑写一个`auth.py`文件来处理用户的注册和登录请求，将其放在项目的`routes`文件夹下。  
- 在`auth.py`文件中，应该是一些视图函数的实现，但是作为一个简单的函数，没有办法通过路由跟html关联起来，所以需要路由，在python中路由的具体实现是通过[装饰器](/python/decorator.md)来实现的，装饰器本质上就是一个函数，而flask中的这个函数就是`.route()`。使用flask开发web应用都需要先实例化一个flask对象，通常命名为`app`，即`app = Flask(__name__)`，而这个对象就有一个`.route()`方法。  
- 以上路由的实现方法仅限于简单的应用，除了实现注册和登录之外，还需要有聊天对话的界面，同样也需要对应的html文件和后端处理函数文件，所以引入蓝图的概念，可以将不同的后端函数分开在不同的文件中，最后再通过蓝图将这些文件注册到flask对象中。  

简单对比一下引入蓝图前后的代码结构：    
原来只有一个app对象：  
```python 
# /__init__.py
from flask import Flask, render_template
app = Flask(__name__)

@app.route('/')
def index():
    return render_template('index.html')

@app.route('/auth/register')
def register():
    ... # 处理注册逻辑
    return render_template('register.html')
```
在路由文件中引入蓝图之后：  
```python
# /routes/auth.py
from flask import Blueprint, render_template
auth_bp = Blueprint('auth', __name__, url_prefix='/auth')
@auth_bp.route('/register')
def register():
    ... # 处理注册逻辑
    return render_template('register.html') 
```
```python
# /routes/__init__.py 
from flask import Blueprint, render_template

main_bp = Blueprint('main', __name__)
@main_bp.route('/')
def index():
    return render_template('index.html')
```
```python 
# /__init__.py
from flask import Flask
app = Flask(__name__)
from routes.auth import auth_bp
from routes import main_bp

app.register_blueprint(auth_bp)
app.register_blueprint(main_bp)
```
> *该代码不一定完全严谨*。  


对比可以发现，将原本的路由函数分离到不同的文件中，使用蓝图之后，代码结构更加清晰。简单来说，蓝图就是让我们可以将不同的路由函数分离到不同的文件中。当web应用比较复杂的时候就需要使用。  

通过以上过程，可以理解flask中最重要的几个概念，同时也梳理出一个比较完整的flask应用的项目结构的例子：
```plaintext
chatgpt_example/
├── app.py                # Flask应用的入口文件
├── routes             # 路由文件夹
│   ├── __init__.py        # 路由初始化文件
│   ├── auth.py            # 处理用户认证的路由
│   └── chat.py            # 处理聊天相关的路由
├── templates          # 模板文件夹，存放HTML文件，放在该文件夹中，就可以直接使用render_template函数来渲染，而不需要指定过于完整和复杂的路径
│   ├── index.html         # 主页面模板 
│   ├── register.html      # 注册页面模板
│   └── chat.html          # 聊天页面模板
├── static            # 静态文件夹，存放CSS、JavaScript等静态资源
│   ├── css                 # CSS文件夹 
│   │   └── styles.css       # 样式文件
│   └── js                  # JavaScript文件夹
│       └── scripts.js       # 脚本文件
```


