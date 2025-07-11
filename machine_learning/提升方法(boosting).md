# 提升方法
>本文主要为《统计学习方法——李航 第二版》的学习笔记。  

Boosting是机器学习领域中的一种算法。其基本思想是通过改变训练样本的权重，学习多个分类器，并将这些分类器进行线性组合。往往boosting会结合决策树算法实现提升树算法。  
## 提升算法的由来  
在概率近似正确（probably approximately correct，PAC）学习的框架中，一个概念（一个类），如果存在一个多项式的学习算法能够学习它，并且正确率很高，那么这个概念就是强可学习的，如果正确率略高于随机猜测，那么就是弱学习的。并且两者其实后来被证明是等价的。而发现弱学习算法比强学习算法要简单的多，为了将找到的弱学习算法提升（boost）为强学习算法，从而提出了boosting。  
> 关于理解上面这段话中的一些关键词汇，可能要从了解*P=NP?问题*以及算法的*时间复杂度*。 ---未完待续---- 

## AdaBoost
对于一个分类问题，提升算法基本策略在于使用数据集确定比较粗糙的分类规则（弱分类器），通过**改变训练数据的概率分布**，得到一系列的弱分类器，然后**组合这些弱分类器成为一个强分类器**。  
而AdaBoost具体地是**通过提高前一轮被弱分类器分类错误的样本的权重值**来改变数据概率分布；  
其最后弱分类器的组合策略为加权多数表决方法：**加大分类误差小的分类器的权重并减小分类误差大的弱分类器的权重**。   
这里希望简单地通过语言文字，少用公式描述AdaBoost算法过程： 
- 训练数据集有N个样本，首先个每个样本一个权重值$w=\frac{1}{N}$。这个权重值集合记为$D$
- 开始迭代计算（迭代起点），对于当前权重集合$D$，训练得到基本分类器$G(x)$，并且计算其在训练集上的误差e
- 当前迭代次数为m，根据误差计算出$G_m(x)$的系数$a_m=\frac{1}{2}log\frac{1-e_m}{e_m}$ 
- 更新权重值集合$D$，我们只看其中一个样本$w_{m+1,i}=\frac{w_{m,i}}{Z_m}exp(-\alpha_my_iG_m(x_i))$，其中$Z_m$是规范化因子，$Z_m=\sum{w_{m,i} exp(-\alpha_my_iG_m(x_i))}$，规范化因子的作用就是保证$\sum_{i=1}^N{w_{m+1, i}}=1$
- 接着回到迭代起点重复迭代过程，得到m个基本分类器$G_m(x)$
- 将多个基本分类器组合，得到最终分类器：$G(x)=sign(f(x))=sign(\sum_{m=1}^M{a_m G_m(x)})$
以上算法实现的最重要的效果就是可以加大被前一个基本分类器分类错误的样本的权重。

## 提升树 GBDT(gradient boosting decision tree)