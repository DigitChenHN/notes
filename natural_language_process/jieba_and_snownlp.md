# 简介
这是两个python的库，常见于中文文本的文本分析任务中，主要能够实现分词、词性标注、情感分析等功能。  

# 分词 
分词是大多数文本分析任务的基础，尤其对于中文文本。与英文不同的是，不同的词在中文表达中是连在一起的，这对于计算机或者算法来说都是非常麻烦的，因为句子之所以可以表达近乎无限的意思，靠的就是只能表达单一意思（当然这个说法比较绝对哈）且数量有限的单词通过排列组合，才形成了近乎无限的语义空间。所以想要拆解语言语义，至少要将文本拆分成单词。  
jieba和snownlp都有分词的功能，但是jieba的分词效果较好。  

## jieba分词函数`jieba.cut()`
```python 
import jieba 

# 精确模式分词
words = jieba.cut("中华人民共和国是一个伟大的国家")
print("/".join(words))
# 输出: 中华人民共和国/是/一个/伟大/的/国家

# 全模式分词
words = jieba.cut("中华人民共和国是一个伟大的国家", cut_all=True)
print("/".join(words))
# 输出: 中华/华人/人民/共和/共和国/和国/国是/一个/伟大/的/国家

# 搜索引擎模式分词
words = jieba.cut_for_search("中华人民共和国是一个伟大的国家")
print("/".join(words))
# 输出: 中华/人民/共和/国是/中华人民/共和国/人民共和国/是/一个/伟大/的/国家
```

该函数返回的是一个生成器对象，还有一个函数`jieba.lcut()`，返回的是一个列表。  
jieba分词的原理据称基于统计的，并且涉及了图论和动态规划。[文章](https://zhuanlan.zhihu.com/p/245372320)做了很好的解释和演示。大概理解的话：这个库内部是有一个统计词典，该词典的结构大概如下： 

```
孟玉楼 3 nr

斗蓬 3 ns

铁法官 3 n

羿射九日 3 nr

占金丰 4 nr
```
第一列为对应的词，第二列为词频，第三列为词性。  
分词程序开始运行时，会通过统计词典生成一个前缀词典。前缀词举例：比如统计词典中的词“羿射九日”，对应的前缀词典中的词就是“羿”、“羿射”、“羿射九”和“羿射九日”。
比如统计词典中是：
```
羿 12
射 121
九日 123
羿射九日 12
```

生成的前缀词典就是：
```
羿 12
射 121
羿射 0
羿射九 0
九日 123
羿射九日 12
```
到此为止就可以有一个感觉，即统计词典是为了划分词汇而提供的一个工具，它大概能够实现的功能就是告诉程序“羿”比“羿射九”更有可能是一个词。  
之后就是对句子建模成一个图，然后做动态规划，这一部分参看上面提到的文章。这里的重点是，如果某词统计词典中没有，那么大概率会影响词的划分效果。比如“甄远道”这个词（不好意思最近在看甄嬛传:rofl:），在统计词典中肯定会有一个词是“甄”，但是大概不会有一个词“甄远”以及“甄远道”，这样的话这两个词在前缀词典中的词频就是0，从而应该会影响后续动态规划算法不会将这三个字划分为一个词。（上述文章中提到的HMM算法似乎能够对未登录的词进行识别）  

所以jieba中提供了一个方法可以向词典中加入新词。  
```python 
jieba.add_word('甄远道')
```  
或者  
```python
jieba.load_userdict('userdict.txt')
```  
这个userdict.txt文件就是自定义的词典，格式和统计词典是一样的。这个dict并不会覆盖原本的词典。    
```python
jieba.set_dictionary('data/dict.txt.big')
```
重新设置内置词典。  

结合上面的内容，结合jieba作者的[回复](https://github.com/fxsjy/jieba/issues/14)，可以更好地理解分词的原理。  

## 停词表
停词表就是将分词好后的列表中不需要的词删除。  
最简单的方法就是用列表推导式，遍历分词然后比对：  
```python 
stopwords_file = 'stopwords.txt'
with open(stopwords_file, 'r', encoding='utf-8') as f:
    stopwords = [line.strip() for line in f]

cut_list = jieba.lcut(sentence)
cut_list = [cut for cut in cut_list if cut not in stopwords]
```
stop_words这样的数据经常用`set()`进行存放，因为`set()`中的元素不能重复。  

```python
stopwords_file = 'stopwords.txt'
with open(stopwords_file, 'r', encoding='utf-8') as f:
    stopwords = {line.strip() for line in f} 

cut_list = jieba.lcut(sentence)
cut_list = list(filter(lambda x: x not in stop_words, cut_list))
```

## 词性分析  
一个词是什么词性其实在jieba库的统计词典中其实已经做了标注了。对于分好的词，如果我们想做下一步统计分析，往往除了使用停词表进行筛外，也可以通过词性进行筛分。`jieba.posseg`提供了词性分析的方法。  
```python
import jieba.posseg as pseg 
wordpair = pseg.lcut("中华人民共和国是一个伟大的国家")
```
`lcut()`函数同样返回的是一个列表，其中的每一个元素是一个pair对象，对象中包含两个属性，分别为`.word`和`.flag`，分别表示词和词性。  
jieba和snownlp的分词预料很多电商评论，实测在对小说进行分词的时候效果并不好，因为小说有大量的特有词汇，需要开启HMM模式以发现未登录的词汇，这样才有可能将“华妃”这样的词划分出来并且进行词性标注。其次就是内置词典本身就是存在词性标注错误，譬如“明白”，“小姐”这两个词都被如果想要消除这些词汇的词性错误，可以手动的编写一个userdict.txt自定义词典并载入jieba。
```
明白 11111 v
小姐 11111 r
```
***注意 自定义词典不要用Windows记事本保存，这样会加入BOM标志，导致第一行的词被误读***

## 关键词提取
### 基于TF-IDF算法的关键词提取  
`jieba.analyse.extract_tags(sentence, topK=20, withWeight=False, allowPOS=())`。  
sentence为待提取的文本；allowPOS参数为指定词性。  
常用的词性有：人名nr；动词v；名词n；地名ns；nt机构团体；vd副动词；a形容词；ad副形词；r代词

### 基于TextRank算法的关键词提取
`jieba.analyse.textrank(sentence, topK=20, withWeight=False, allowPOS=('ns', 'n', 'vn', 'v'))`。  

## 情感分析
情感分析可以使用`snownlp.snowNLP` 这个类。snownlp这个库用来做情感分析时的训练数据主要时电商评论，因此对于小说人物的语言情感分析效果比较差。  
### 基本用法  
```python
from snownlp import SnowNLP

s = SnowNLP(u'这个东西真心很赞')

s.words         # [u'这个', u'东西', u'真心',
                #  u'很', u'赞']

s.tags          # [(u'这个', u'r'), (u'东西', u'n'),
                #  (u'真心', u'd'), (u'很', u'd'),
                #  (u'赞', u'Vg')]

s.sentiments    # 0.9769663402895832 positive的概率

s.pinyin        # [u'zhe', u'ge', u'dong', u'xi',
                #  u'zhen', u'xin', u'hen', u'zan']

s = SnowNLP(u'「繁體字」「繁體中文」的叫法在臺灣亦很常見。')

s.han           # u'「繁体字」「繁体中文」的叫法
                # 在台湾亦很常见。'
```
基本功能可以实现文本分词、词性划分、情感分析。使用这个类时只需要输入一个要分析的文本的Unicode编码格式的字符串。  
比如，对于小说人物对话情感分析的应用场景，可以将人物的话逐一计算positive的概率，然后计算平均值；也可以考虑将所有对话提取出来，使用`' '.join(sentences)`将其合并一个字符串。  
### 情感词典  
由于这个库的情感分析模型是通过电商评论数据训练出来的，因此对于其他领域的文本比如小说的分类效果可能不太好。有两种方法可以解决，其一就是直接使用[情感词典](https://github.com/ppzhenghua/SentimentAnalysisDictionary)。  
一个现成的情感词典的结构大致如下：
```
沉醉于	-0.4
要	0.48549295774647866
暗查	-0.4
陶醉在	1.95
臭老九	-0.7
齐唱	1.0
宾至如归	1.0
佛罗里达	1.2
俯视	1.0499999999999998
纳西	-0.48
土音	1.4
容量	0.6
宠辱不惊	1.6
```
使用方法就是直接在对文本进行分词，并在情感词典中查找该词汇，直接对分数进行加和。  
```python
import pandas as pd 
df = pd.read_csv('sentiment_dict.txt', sep='\t', names = ['key', 'score']) 

key = df['key'].values.tolist()
score = df['score'].values.tolist()

def getscore(line):
    segs = jieba.lcut(line)
    score_list = [score[key.index(seg)] for seg in segs if seg in key]
    return sum(score_list) 
```

> `.index()`是list的方法，返回第一个匹配元素的索引  

### 训练情感分类模型  
snownlp也提供了训练分类模型的方法。训练情感模型需要正负情感两个词语文件。在[这个GitHub仓库](https://github.com/ppzhenghua/SentimentAnalysisDictionary/tree/main)同样有提供。  

```python
from snownlp import sentiment  
sentiment.train('neg.txt', 'pos.txt') 
sentiment.save('sentiment.marshal')  
```
这样训练好的文件就保存在`seg.marshal`中了，之后修改`snownlp/sentiment/__init.py`中的`data_path`指向刚训练好的文件即可。  

以下是`snownlp/sentiment/__init.py`源文件：  
```python 
# -*- coding: utf-8 -*-
from __future__ import unicode_literals

import os
import codecs

from .. import normal
from .. import seg
from ..classification.bayes import Bayes

data_path = os.path.join(os.path.dirname(os.path.abspath(__file__)),
                         'sentiment.marshal')

class Sentiment(object):

    def __init__(self):
        self.classifier = Bayes()

    def save(self, fname, iszip=True):
        self.classifier.save(fname, iszip)

    def load(self, fname=data_path, iszip=True):
        self.classifier.load(fname, iszip)

    def handle(self, doc):
        words = seg.seg(doc)
        words = normal.filter_stop(words)
        return words

    def train(self, neg_docs, pos_docs):
        data = []
        for sent in neg_docs:
            data.append([self.handle(sent), 'neg'])
        for sent in pos_docs:
            data.append([self.handle(sent), 'pos'])
        self.classifier.train(data)

    def classify(self, sent):
        ret, prob = self.classifier.classify(self.handle(sent))
        if ret == 'pos':
            return prob
        return 1-prob


classifier = Sentiment()
classifier.load()


def train(neg_file, pos_file):
    neg_docs = codecs.open(neg_file, 'r', 'utf-8').readlines()
    pos_docs = codecs.open(pos_file, 'r', 'utf-8').readlines()
    global classifier
    classifier = Sentiment()
    classifier.train(neg_docs, pos_docs)


def save(fname, iszip=True):
    classifier.save(fname, iszip)


def load(fname, iszip=True):
    classifier.load(fname, iszip)


def classify(sent):
    return classifier.classify(sent)
```
本质上是训练了一个贝叶斯分类器。  
初次进行自定义贝叶斯模型训练后，使用`save`方法将分类器保存在任意位置。  
```python
import snownlp 
snownlp.sentiment.train('NTUSD_positive_simplified.txt', 'NTUSD_negative_simplified.txt')
snownlp.sentiment.save('my_model/sentiment.marshal')
```
下次重新导入snownlp的时候，根据源码中以下的内容，模型会默认加载`snownlp/sentiment/__init.py`同路径下的`sentiment.marshal`文件。  
```python
data_path = os.path.join(os.path.dirname(os.path.abspath(__file__)),
                         'sentiment.marshal')

class Sentiment(object):
    
    ## ····
    def load(self, fname=data_path, iszip=True):
        self.classifier.load(fname, iszip)
    ## ····

classifier = Sentiment()
classifier.load()
```
这时候我们就需要手动加载模型文件：  
```python
snownlp.sentiment.load('my_model/sentiment.marshal')

sentence = "毒打一顿"
score = SnowNLP(sentence).sentiments 
print(score)
```
除了情感分析可以训练之外，分词和词性标注也提供了训练方法。  
```python
from snownlp import seg
seg.train('data.txt')
seg.save('seg.marshal')
# from snownlp import tag
# tag.train('199801.txt')
# tag.save('tag.marshal')
```
