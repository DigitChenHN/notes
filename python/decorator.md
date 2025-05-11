# 装饰器
如果我们定义了一个计算两个数的和的函数。
```python
def add(x, y):
    return x + y
```
随后我们增加一些功能，计算函数的执行时间。这个时候我们可以考虑修改函数体内部的逻辑
```python
import time
def add(x, y):
    start = time.time()
    result = x + y
    end = time.time()
    print(f"函数执行时间为{end - start}秒")
    return result
```
但是这种修改函数体逻辑的方法在函数本身逻辑比较复杂的情况下会改变代码，降低其可读性。其次也不方便代码的复用，如果其他函数也需要实现同样的功能，又需要修改函数体逻辑。最重要的一点是，软件开发有一个“单一职责”的原则，即一个函数只做一件事情。  
因此可以考虑使用嵌套函数，将原函数放到另一个外函数中，这个外函数接受原函数作为参数：
> 这里使用嵌套函数的逻辑是，我们希望有一个能够输入一个函数A，然后输出一个新的函数B的函数C，这个函数C就需要将函数作为参数，并且这个函数C要干的事情就是定义一个函数（修改一个函数），所以函数C的内部要做的事情就是定义函数，所以自然而然的就要使用  
```python
def decorator(func):
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        end = time.time()
        print(f"函数执行时间为{end - start}秒")\
        return result
    return wrapper
```
调用decorator函数，得到一个新的函数wrapper，这个函数接收的参数跟原函数是一样的（不管原函数是add还是其他任何函数，`*args`和`**kwargs`保证了这一点），这个wrapper函数将返回跟函数一样的结果`result`，同时添加了计算函数执行时间的功能。 
```python
add = decorator(add)
```
这样一来，函数add就被扩展了新的功能，最关键的是有了这个decorator函数，我们可以为其他任何函数都添加计算函数时间的功能。  
为了方便使用，python为这种行为设置了语法糖，使得程序员在实现上述的功能时，程序更加简洁。  
> 语法糖并不会改变语言原有的功能，只是为了方便程序员使用而提供的一种语法。

首先仍然是需要定义好装饰器函数，但是在使用时，可以直接使用“@+装饰器函数名”的方式。  
```python 
@decorator
def add(x, y):
    return x + y
```
这种语法糖其实就是将add作为参数传给decorator这个函数，仅仅是这样而已。  
