# 简介
Pandas是python的一个库，其主要用途是对二维表格进行操作。经常用于数据分析领域。 

# Pandas中最基本的类——`Dataframe`
Pandas中核心的类是Dataframe，是用来描述一个二维表格的类，可以储存表格类型的数据，并且实现各种计算和操作。

# 二维数据表常用的操作对应以及pandas中的实现  
> 以下内容是依据个人使用频率，并结合官方文档介绍一些常用的操作，目的是为了系统地构建个人的知识体系，因此可能会加入比较多的个人理解。如果比较熟悉python和面对对象编程，直接参看官方文档更方便。
> 个人认为学习第三方库来解决问题，最重要的是知道了解**传入的参数**和**返回值**。

## `Dataframe.columns`和`Dataframe.index`  
一般而言，数据表的行代表每一个样本，列代表每个样本的各个特征。  
Dataframe的属性`.columns`常用来获取列名，返回的是一个`Index`对象。  
``` python
>>> df = pd.DataFrame({'A': [1, 2], 'B': [3, 4]})
>>> df
     A  B
0    1  3
1    2  4
>>> df.columns
Index(['A', 'B'], dtype='object')
>>> print(type(df.columns))
<class 'pandas.core.indexes.base.Index'>
```
相应的，一个Dataframe的`.index`属性则是包含行名（行标签）的`Index`对象。 

## `Dataframe.dtypes`
一般而言，pandas中的对象主要是用来存储数据的，所以基本上都会有相应的`dtype`属性，比如上例的`df.columns`，它的数据类型是*object*。  
而`DataFrame.dtype`会返回一个`pandas.Series`，这个Serise的`.index`属性就是原Dataframe中每一列的列名，每个`index`对应的值就是对应列的`dtype`。  
>这里提到的`pandas.Serise`是pandas中另一个比较核心的类。Dataframe是用来描述二维表格的，Series则描述一维表格，可以看成是只有一列的Dataframe。由于只有一列，所以没有`.columns`属性，只有`.index`属性。  

也就是说，Dataframe中默认每一列中所有的数据都应该是相同的`dtype`。如果将数据表仅仅当作数据存储器，那么同一列中的数据可以是各种不同的类型；但对于大多数使用二维表格（线性表格）来储存的数据对象，往往会约定每一行代表一个样本，而每一列代表一个特征（从而表示数据之间的关系），因此同一个列中的值理论上应该是一个类型。  
但是往往收集到的原始数据的数据格式也并不统一，因此在使用`pandas.read_csv()`函数读取一个csv文件并构建一个Dataframe对象时，部分应该是数值类型的列的`dtype`被自动解析成`object`。这时候可以使用`pandas.to_numeric()`函数转化成数值类型。   
`pandas.to_numeric(arg, errors='raise', downcast=None, dtype_backend=<no_default>)`   
这个函数默认会将数据转换成*float64*或者*int64*。其中`errors`参数，默认的`raise`选项意味着如果数据无法转换成数值型，将会引发报错；还有一个常用的选项`coerce`，代表着非法的值将会被设置为NaN。

## `Dataframe.values`
使用pandas处理数据很多时候都会需要结合numpy进行数据运算。  
这个属性将会返回一个dataframe的numpy格式，也就是**返回**一个二维的`numpy.ndarray`。  
但是现在官方推荐使用`Dataframe.to_numpy()`这个方法，同样**返回**的是一个`numpy.ndarray`。  

## 索引  
>索引是处理数据使非常重要的操作。   
### 标准python索引  
`[]`是python中的索引操作符，而在pandas中同样可以使用。  
比如，`Dataframe['列名']`用来索引某一列的数据，**返回**的是一个`pandas.Serise`对象；如果传入的是列名的列表，**返回**的则是一个`pandas.Dataframe`。  
>使用`[]`索引实际上是调用对象的`__getitem__`方法。  

### `.loc`和`.iloc`方法
*首先注意这两个索引方法的调用用的是[]，而非传统的()。*   
`.loc`方法是“label based”的，传入的参数作为标签用来索引。   
而`.iloc`方法则是“position based”的，也就是说传入的应该是整数，并且这些整数是作指示位置的作用。两者的区别可见下面的列子：    
```python 
In [18]: df[['A', 'B']]
Out[18]: 
                   A         B
2000-01-01  0.469112 -0.282863
2000-01-02  1.212112 -0.173215
2000-01-03 -0.861849 -2.104569

In [19]: df.iloc[:, [1, 0]] = df[['A', 'B']]

In [20]: df[['A','B']]
Out[20]: 
                   A         B
2000-01-01 -0.282863  0.469112
2000-01-02 -0.173215  1.212112
2000-01-03 -2.104569 -0.861849
```
在以上的例子当中，传入`.iloc[]`的第二个参数`[1, 0]`意味着数据表的第二列和第一列，即将数据表的第一列和第二列的数据调换。而如果考虑通过`.loc[]`实现，则需要：
```python
In [13]: df[['A', 'B']]
Out[13]: 
                   A         B
2000-01-01 -0.282863  0.469112
2000-01-02 -0.173215  1.212112
2000-01-03 -2.104569 -0.861849

In [14]: df.loc[:, ['B', 'A']] = df[['A', 'B']]

In [15]: df[['A', 'B']]
Out[15]: 
                   A         B
2000-01-01 -0.282863  0.469112
2000-01-02 -0.173215  1.212112
2000-01-03 -2.104569 -0.861849
```
可以发现使用`.loc[]`并没有使得两列数据调换，原因在与使用`.loc[]`时，pandas会对齐Dataframe的所有AXES。也就是说会将列名进行匹配对其之后，在进行值的分配，所以名为A列的数据仍然是原来A列的数据。  
### 索引单个值`.at[]`
`.at[]`方法和`.loc[]`方法类似是基于标签的，实际上`.loc[]`也可以获得单个的值，但是如果索引单个的值，一般会使用`.at[]`方法。与之对应的有`.iat[]`，即基于**位置**的索引单个值的方法，需要传入的参数是用来指示位置的整数。  


事实上，索引在实际的数据分析过程中，往往是结合条件判断，从而实现某些位置上的数据值的获取、修改和删除。也就说往往会给索引方法传入Boolean，从而获取某些符合条件的值。  
例如想将所有A列中值等于9的样本删除：  
```python
df = df[~(df['A'] == 9)]
```
以上语句就向索引方法传入了Boolean值——`~(df['A'] == 9)`。  
*此外，修改某些值也可使用`Dataframe.replace(to_replace=None, value=<no_default>,...)`方法，`to_replace`参数可接收要被替换的数据，可以是str、list、dict等等，`value`参数指定用来替换`to_replace`的值。  

## 对数据表进行操作
对数据表进行操作，往往首先需要考虑“按行”还是“按列”操作，因此这类函数一般有一个参数`axis=0`，参数值为0时代表“按行”，参数值为1时代表“按列”。  
- 去除nan值，`DataFrame.dropna(*, axis=0, how=<no_default>, thresh=<no_default>, subset=None, inplace=False, ignore_index=False)`，其中*subset*可以用来指定列名从而选定查找nan值的列；*how*可以是`any`或者`all`，分别代表只要存在nan值就删除整行，或者必须所有列都存在nan值才删除整行。  
- 去除重复值删除重复的行，`DataFrame.drop_duplicates(subset=None, *, keep='first', inplace=False, ignore_index=False)`

## 一些复杂的操作