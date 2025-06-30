# 简单说明
flask是一个基于python的轻量级web框架，其中包含了大量的开发一个web应用常用的功能和扩展，比如一般一个web应用都需要提供给用户注册、登录、数据存储，而flask中就有大量的现成的函数。个人认为真正要理解这些框架的使用，还是需要使用框架去开发一个简单的应用，单纯看文档或者某些系统的教程，很容易陷入大量的术语迷宫中，[菜鸟教程](https://www.runoob.com/flask/flask-tutorial.html)中已经介绍的比较通俗了，但是没有web开发基础，不能结合完整的实际案例的话，仍然容易陷入术语陷阱中。  

# 背景
现在假设我们要开发一个简单的web应用，核心的功能就是为用户提供一个跟大语言模型聊天的界面。  
假设我们的网站域名叫做`chatgpt.example.com`。
- 当用户进入网站的时候，首先展示的是主页面，由于此时是未登录的状态，此时应该展示一个html页面（就命名为index.html吧），页面中有一个登录和一个注册的按钮，这个html作为`主页面`。  
- 当用户点击注册按钮的时候，就要能够跳转到`注册页面`，假设我们希望地址跳转到`chatgpt.example.com/auth/register`，那么这个注册页面也需要有一个html文件（就命名为register.html吧）来展示注册表单。  
    > *注册表单可以让用户填入用户名、密码等信息，然后用户点击注册之后，信息会被提交到服务端，后端的代码会将用户信息存储到数据库中*。  


那么为了实现以上过程：
- 首先需要展示相应的html文件，这些html文件我们可以考虑统统放到项目的`templates`目录下。
- 然后点击某个页面中的按钮，需要与数据库交互之后再跳转到另外一个html页面上，如果是单纯的html页面，那么也许可以考虑给这个元素添加一个`href`属性，并指向另一个html文件（填入该html的路径）。但是在web应用中，是需要更复杂的逻辑的，比如与后端交互之后再展示另一个html页面，所以在flask框架下，`href`属性的值应该是一个**路由**地址，而不是简单地指向某个文件的路径。    
- **路由**的作用就是将用户的请求映射到相应的处理函数上，在flask中，处理函数就是一个普通的python函数，也叫**视图函数**，通常接收一个请求对象作为参数。比如对于注册和登录，可以考虑写一个`auth.py`文件来处理用户的注册和登录请求，将其放在项目的`routes`文件夹下。  
- 在`auth.py`文件中，应该是一些视图函数的实现，但是作为一个普通的python函数，是没有办法跟html关联起来，所以需要路由，在flask中，路由的具体实现是通过[装饰器](/python/decorator.md)来实现的，装饰器本质上就是一个函数，而flask中的这个函数就是`.route()`。使用flask开发web应用都需要先实例化一个flask对象，通常命名为`app`，即`app = Flask(__name__)`，而这个对象就有一个`.route()`方法。  
- 以上路由的实现方法仅限于简单的应用。在我们的案例中，除了实现注册和登录之外，还需要有聊天对话的界面，这同样也需要对应的html文件和后端处理函数文件。而为了保证项目的可读性，使不同功能的函数分开在不同的文件中，这里引入蓝图的概念，通过蓝图可以将将这些不同的文件注册到同一个flask对象中。  

简单对比一下引入蓝图前后的代码结构：    
1. 原来只有一个app对象：  
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
2. 在路由文件中引入蓝图之后：  
```python
# /routes/auth.py
from flask import Blueprint, render_template
auth_bp = Blueprint('auth', __name__, url_prefix='/auth')
@auth_bp.route('/register')
def register():
    ... # 处理注册逻辑
    return render_template('register.html') 
```
<anchor id="blueprint"></anchor>

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
> *该代码不一定完全严谨，主要是为了展示路由和蓝图的实现*。  


对比可以发现，将原本的路由函数分离到不同的文件中，然后使用蓝图，使得代码结构更加清晰。简单来说，蓝图就是让我们可以将不同的路由函数分离到不同的文件中。当web应用比较复杂的时候就需要使用。  

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
其中，想templates、routes、static等文件夹都是flask框架的约定俗成的文件夹名称，flask会自动识别这些文件夹中的文件。比如，templates中的html文件可以直接使用`render_template`函数来渲染，而不需要指定过于完整和复杂的路径。  

# 细碎要素
## jinja
jinja是flask中使用的模板引擎，在一个flask框架实现的项目中，在templates文件夹下的这些html就是所谓的**模板文件**，其中可能会出现一些部署与html或者javescript的语法，比如：{{ variable }}，这就是jinja的语法。通过这些语法，就可以将python中的一些变量（数据）渲染到html中。  
jinja常用的语法有：
- `{{ variable }}`：渲染变量，比如`{{ username }}`会将`username`变量的值渲染到html中。
- `{% if condition %} ... {% endif %}`：条件语句，比如`{% if user.is_authenticated %} ...{% endif %}`会根据`user.is_authenticated`的值来决定是否渲染其中的内容。
- `{% for item in items %} ... {% endfor %}`：循环语句 ，比如`{% for message in messages %} ... {% endfor %}`会遍历`messages`列表中的每个元素，并渲染其中的内容。  

## 模板文件  
在flask项目中，模板文件通常放在`templates`文件夹下。之所以叫模板文件，是因为这些html并不是静态地展示固定的内容，而是可以根据情况展示不同的内容，并且不同的html文件之间可能有一些公共的部分， 比如导航栏、页脚等，这些也可以使用jinja的语法来实现。  
比如我们希望整个网页的导航栏是一样的，就可以在`templates`文件夹下创建一个`base.html`文件，然后在其他的html文件中继承这个`base.html`文件。  
```html
<!-- templates/base.html -->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{% block title %}ChatGPT Example{% endblock %}</title>
    <link rel="stylesheet" href="{{ url_for('static', filename='css/styles.css') }}">
</head>
<body>
    <nav>
        <ul>
            <li><a href="{{ url_for('main.index') }}">Home</a></li>
            <li><a href="{{ url_for('auth.register') }}">Register</a></li>
            <li><a href="{{ url_for('chat.chat') }}">Chat</a></li>
        </ul>
    </nav>

    <div class="container">
        {% block content %}{% endblock %}
    </div>

    <!-- 自定义JS -->
    {% block extra_js %}{% endblock %}
</body>
</html>
```
以上是一个简单的带有导航栏的`base.html`文件，其中使用了jinja的语法来渲染标题和链接。其中固定的内容是<head>和<nav>部分，而还有一部分内容用`{% block title %}{% endblock %}``{% block content %}{% endblock %}`和`{% block extra_js %}{% endblock %}`来占位，这样在其他的html文件中就可以继承这个`base.html`文件，并在其中填充具体的内容。  
比如一个注册页面的html文件想要直接复用导航栏，就可以这样写：  
```html
<!-- templates/register.html -->
{% extends 'base.html' %}

{% block title %}Register{% endblock %} 

{% block content %}
<!-- 属于注册页面的内容 -->


{% endblock %}

{% block extra_js %}
<script src="{{ url_for('static', filename='js/register.js') }}"></script>
{% endblock %}

{% endblock %}
```
其中，最开头的`{% extends 'base.html' %}`表示这个html文件继承自`base.html`文件，然后在`{% block title %}`、`{% block content %}`和`{% block extra_js %}`中填充属于注册页面的内容。这样就可以实现多个页面之间的内容复用，减少代码重复。  

## 一些常见的jinja语法
### `url_for`函数
前面说了，在html文件中要关联到后端的处理函数，需要使用路由，那么这个路由地址应该如何写呢？在flask中，通常使用`url_for`函数来生成路由地址。这个函数接收一个视图函数的名称作为参数，并返回该视图函数对应的路由地址。  
比如在`base.html`中，我们可以这样写：  
```html
<a href="{{ url_for('main.index') }}">Home</a>
```
其中的`main.index`就是视图函数的名称，表示`main`蓝图下的`index`视图函数。而这个`mian`名称是通过[蓝图定义](#blueprint)确定的，即传入`Blueprint`这个类的构造函数的第一个参数。  


### `render_template`函数  
`render_template`是flask模块中的一个函数，和jinja模板引擎配合使用。其函数签名如下： 
```python
def render_template(
    template_name_or_list: str | Template | list[str | Template],
    **context: Any
) -> str
```
其中第一个参数`template_name_or_list`是template文件夹下的模板文件的路径，第二个参数**可变长度字典参数**可以接收任何关键字参数，这些参数都可以在html模板文件中使用jinja语法来渲染。  
比如在`chat.py`文件中，我们可以这样写：  
```python
# routes/chat.py 
from flask import Blueprint, render_template, request
chat_bp = Blueprint('chat', __name__, url_prefix='/chat')
@chat_bp.route('/')
def chat():
    # 假设我们有一些聊天记录需要展示
    messages = [
        {"user": "Alice", "message": "Hello!"},
        {"user": "Bob", "message": "Hi there!"},
    ]

    return render_template('chat.html', messages=messages)
``` 
之后就可以在`chat.html`文件中使用jinja语法来渲染这些消息了：  
```html
<!-- templates/chat.html -->
{% extends 'base.html' %}

{% block title %}Chat{% endblock %}

{% block content %}
<div class="chat-container">
    {% for msg in messages %} 
        <div class="message">
            <strong>{{ msg.user }}:</strong> {{ msg.message }}
        </div>
    {% endfor %}
</div>
{% endblock %}
```
在jinja的这些括号中，完全使用的就是python的语法，比如上述例子中的messages就是`chat`视图函数中定义的一个列表。  

