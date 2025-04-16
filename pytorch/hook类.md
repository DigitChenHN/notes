# Hook
Hook是钩子的意思，不仅仅在pytorch中有，而是一个计算机编程中的技术概念，仅以个人目前在pytorch中接触到hook来理解，就是能够在计算机程序的数据流或者执行指令流中“钩住”住某些东西的技术。  
## Hook 在pytorch中的具体使用方法
一个pytorch的hook可以用来记录一个pytorch.nn.Module的向前传播输入和输出。  
使用的方法是调用nn.Module.register_forward_hook方法:  
- 该方法的第一个参数可以是一个hook函数，也可以是一个具有__call__方法的hook对象的实例，关键在于其callable。
- hook函数的参数为module、module的输入、module的输出。
- hook函数注册为某个`nn.Module`上的钩子之后，当该module调用forward方法时，会将module对象本身、输入、输出自动依次传递给hook的位置参数。 
- 因此为了得到module的输入和输出，hook函数具体的功能就是将这些传入的东西记录下来，比如这个hook函数是一个hook对象的方法，那么这个hook函数需要做的就是将这些输入赋值给hook对象的属性，这样一来在forward方法执行完之后，就可以通过hook对象获得模型的输入和输出了。  
最后，nn.Module.register_forward_hook方法**返回**一个hook句柄handle，使用handle.remove()将注册在Module中的hook删除。  
1. 定义一个Hook类：  
```python
class Hook(object):
    def __init__(self):
        self.module_name = []
        self.features_in_hook = []
        self.features_out_hook = []

    def __call__(self, module, fea_in, fea_out):
        print("hooker working", self)
        self.module_name.append(module.__class__)
        self.features_in_hook.append(fea_in)
        self.features_out_hook.append(fea_out)
        return None
``` 

2. 在forward过程中使用hook（其中`model`是一个pytorch模型；`loader`是一个pytorch的`DataLoader`）:  
```python 
hook = Hook()
handle = model.features.register_forward_hook(hook)
# 特征提取层（卷积层）的输入，也是模型的输入，是一个形状为(batch_size, channels, height, width)的tensor，module的输入被添加到hook的时候，tensor会被放入一个元组
# 特征提取层（卷积层）的输出，形状为(batch_size, channels, height, width)的tensor，而module的输出被hook使用时，不会被放入一个元组当中
for sample in iter(loader):
    model(sample['image'])
handle.remove()
```
以上例子中，为model的features子模块注册了hook，因此每一次for循环中，model的features子模块的输入和输出都会被hook记录下来并保存到hook对象对应的属性中，在本例中，就是分别被放到`features_in_hook`和`features_out_hook`这两个列表中。 

3. 调用捕获的数据：  
```python 
inputs = []
for input in hook_salt.features_in_hook:
    inputs.append(input[0])
feature_maps = []
for feature_map in hook_salt.features_out_hook:
    feature_maps.append(feature_map)
```  

在pytorch框架下使用hook捕捉模型输入与输出时，尤其要注意的是，输入总是被放入元组中再传递给hook的。具体到本文的例子，就是输入的张量tensor被放入一个元组`(tensor,)`之后再被传递给hook类的`__call__`方法的`fea_in`这个形参。其根本原因，可以看pytorch源码中`nn.Module`的`forward`方法的函数签名：`def _forward_unimplemented(self, *input: Any) -> None:`。根据源码中的这个函数签名可以发现其参数传递采用的是`*input`，这种形式的形参可以接收任意数量的实参，然后将它们打包为一个元组，再供函数体内使用。这是pytorch中为了兼容多输入场景和保证接口统一性而做的设计。需要强调的是，即便是我们自己定义的一个module的类并且重定义了`forward(self, input)`方法，对其注册了hook之后，仍然会传递给hook一个元组。   
***因此使用hook获得输入时要注意将输入从元素中解包。***也就是如上面代码中需要通过`input[0]`来获得input这个元组`(tensor,)`的第一个元素。   
而模块的输出往往由模块作者在`forward`方法的函数中用`return`进行控制，所以一般输出一个tensor，那么hook捕获的就是一个tensor。  
