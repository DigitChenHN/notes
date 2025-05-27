## 简介  
tokenization（词元化）是自然语言处理中的一个重要步骤。严格意义上的tokenization就是将**长文本分割更小的组成然后进行编码**的过程，比如单词（word）、子词（subword）或者字符（character）。  
对于英文文本，由于存在空格，将长文本分割成单词非常简单；但是为了实现对未见单词的处理，也可以将文本分割为子词的，比如`playing -> [play, ##ing]`。  
对于中文文本，[jieba分词](/natural_language_process/jieba_and_snownlp.md)简单介绍了中文的分词工具，其使用的方法可以很好地将文本分割成有意义的单词。  
由于程序只能处理数字，因此将长文本分解成小的基本单元之后，还需要将分出来的单元编码成数字。  
*中文分词的内容在[jieba分词](/natural_language_process/jieba_and_snownlp.md)中有过了解，本文可能重点记录的是一些编码和向量化的概念，[这篇文章](https://zhuanlan.zhihu.com/p/631463712)对现代深度学习的NLP中的tokenization有较为清晰的讲解。*

## 词袋模型 Bag of Words, BoW  
在传统的nlp方法中，词袋法就是一种文本离散表示（用数字表示长文本）的方法。  
比如有模型有一个词汇表：["And", "This", "document", "first", "is", "one", "second", "the", "third"]，其中一共有9基本单元。  
现在要表示句子“This document is the first document”，首先进行简单的分词：["This", "document", "is", "the", "first", "document"]。  
然后构造一个长度为9的文本向量A：[0, 0, 0, 0, 0, 0, 0, 0, 0]，每个元素代表词汇表中的一个单词。遍历原句子分词之后得到的单词，遇到一个单词就将对应的向量元素加1。最终得到的向量A为：[0, 1, 2, 1, 1, 0, 0, 1, 0]。
这种编码的方法有些类似于将one-hot编码。  

使用sklearn的CountVectorizer可以很方便地实现词袋法。  
```python
from sklearn.feature_extraction.text import CountVectorizer

# 示例文本
texts = [
    "This is the first document.",
    "This document is the second document.",
    "And this is the third one.",
]

# 初始化 CountVectorizer
vectorizer = CountVectorizer()

# 拟合并转换文本
X = vectorizer.fit_transform(texts)

# 输出词汇表
print("词汇表：", vectorizer.get_feature_names_out())

# 输出词袋模型矩阵
print("词袋模型矩阵：")
print(X.toarray())

>>> 词汇表： ['and', 'document', 'first', 'is', 'one', 'second', 'the', 'third', 'this']
>>> 词袋模型矩阵：
    [[0 1 1 1 0 0 1 0 1]
    [0 2 0 1 0 1 1 0 1]
    [1 0 0 1 1 0 1 1 1]]
```
`CountVectorizer`是sklearn中用于文本特征提取的一个类，根据这个类的注释可以知道这个类可以用来实现词元的计数： 
> Convert a collection of text documents to a matrix of token counts.  
> This implementation produces a sparse representation of the counts using scipy.sparse.csr_matrix.  
> If you do not provide an a-priori dictionary and you do not use an analyzer that does some kind of feature selection then the number of features will be equal to the vocabulary size found by analyzing the data.  
  
初始化这个类时，有几个重点的参数需要考虑一下：  
- `analyzer`：指定如何将文本分解为特征。
  - 默认是`word`，表示按单词进行分析，单词是通过正则表达式匹配的，即`\w+`，也就是连续的【字母、数字、下划线】。
  - 可以设置为`char`，表示将文本按照字母进行分解。 
  - 还可用传入一个callable对象，可以是一个函数，该函数以文本为输入，返回一个可迭代的特征列表。
- `tokenizer`：指定一个自定义的分词函数。（只有当analyzer参数为`word`的时候才生效，会替代掉原本的分词方法，但是保留预处理和n-gram）。
  - 这个分词函数接收一个字符串作为输入，返回一个字符串列表。譬如可以使用[jieba分词](/natural_language_process/jieba_and_snownlp.md)中提到的`jieba.lcut()`函数。  
  - 区分analyzer和tokenizer这两个参数就要意识到这其中特征和分词的概念。tokenizer用来自定义当`analyzer为`word`时的分词方法；而analyzer是用来指定如何将文本分解为特征的，分词只是做特征提取的一种方法而已。  
  - 譬如如果分析的文本是中文，默认的`analyzer`是`word`，那么就会使用正则表达式`\w+`来进行分词，这将会使得连续的汉字被当作一个词。第一种方法可以考使用`jieba.cut`作为tokenizer参数的值；第二种方法也可以考虑将`jieba.cut`作为analyzer参数的值。 
- `ngram_range`：接收一个元组，用来指定n-gram的范围，默认是(1, 1)，表示只考虑单个单词。可以设置为(1, 2)，表示考虑单个单词和二元组（2-gram）。*关于n-gram的概念在[下节](#n-gram)会有介绍。*

实例化类`CountVectorizer`之后，得到的对象最重要的一个方法就是`fit_transform()`，该方法接收一个`list[str] | ndarray | Iterable`类型的参数，表示要分析的文本数据。该方法会返回一个`scipy.sparse.csr_matrix`类型的对象，表示词袋模型矩阵。  
此外还有一个方法`get_feature_names_out()`，可以用来获取词汇表中的特征名称。返回的数据是一个元素为字符串的ndarray，表示词汇表中的单词。  


词袋法虽然简单易用，但是存在很多缺点：  
1. 词袋法忽略了单词的顺序，"I love you"和"you love I"会被表示成同一个向量。  
2. 词汇向量化之后维度等于词汇表的大小，词向量高度稀疏（很多元素都是0）。  

## n-gram
n-gram的概念有点类似于在[jieba分词](/natural_language_process/jieba_and_snownlp.md)中提到的前缀词。  
n-gram是指在文本中连续出现的n个单词的组合。比如对于句子"This is a pen"，2-gram就是["This is", "is a", "a pen"]，3-gram就是["This is a", "is a pen"]。 
将文本表示为n-gram的好处是可以保留单词的顺序信息。  
如果考虑使用n-gram的词袋模型，首先仍然是构建词汇表。比如如果只有文本"This is a pen"和"This is a book"，那么2-gram词汇表就是["This is", "is a", "a pen", "a book"]。然后构建文本向量，遇到一个n-gram就将对应的向量元素加1。  
```python   
text = [
    "This is a pen",
    "This is a book"
]
vec = CountVectorizer(analyzer='word', ngram_range=(1,2)) 
X = vec.fit_transform(text)

print('词汇表：', vec.get_feature_names_out())

print('文本矩阵：\n', X.toarray())

>>> 词汇表： ['book' 'is' 'is book' 'is pen' 'pen' 'this' 'this is']
>>> 文本矩阵： 
   [[0 1 0 1 1 1 1]
    [1 1 1 0 0 1 1]]
```
在词袋法中使用n-gram的有点是可以在一定程度上保留单词的顺序信息，但是仍然存在维度较高，转换的矩阵稀疏等问题。  

## ngram文本评估  


## TF-IDF 
[关于TF-IDF](https://www.cnblogs.com/Luv-GEM/p/10543612.html)的介绍可以参考这篇文章。  

***以上介绍的是传统nlp中tokenization的一些概念，结合一些机器学习方法可以实现简单的nlp任务比如文本分类等。但是精确的文本生成任务现在主要依靠基于深度学习的方法，深度学习的方法tokenization几乎绑定了embedding向量化的方法。***