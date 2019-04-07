title: "概率学习——朴素贝叶斯算法"
date: 2019-04-07 19:10:50 +0800
update: 2019-04-07 19:10:50 +0800
author: me
cover: "-/images/naive-bayes.png"
tags:
    - Machine Learning
preview: 朴素贝叶斯算法是一种依据贝叶斯定理的分类技术，利用概率原则进行分类，该算法易于构建模型，适用于大规模数据集的分类任务。

---

基于贝叶斯定理的朴素贝叶斯算法是利用训练数据并根据特征的取值来计算每个类别被观察到的概率。这里的“朴素”并不意味着算法有限或者效率低下，而是该算法依赖于一个基本假设，即预测变量具有独立性，也就是说假定类中特定特征的存在与任何其他特征的存在无关。虽然有时该条件看起来很难成立，但是在特定情况下，不同情况之间的依赖关系可能会相互清除，即使朴素贝叶斯分类器的朴素性假设不成立，但是依然能够表现出相当好的性能。

## 一、贝叶斯定理

考虑两个概率事件A和B，使用乘积法则可以将边缘概率P(A)和P(B)与条件概率P(A|B)和P(B|A)相关联：

![](/images/article/bayes1.png)

考虑到交集可以互换，等式的左边是相等的，那么就可以得到贝叶斯定理，如下：

![](/images/article/bayes2.png)

还需要再了解一下似然函数，可以简单的理解为是一种关于统计模型参数的函数。在给定输出x时，关于参数θ的似然函数L(θ|x)(在数值上)等于给定参数θ后变量X的概率：L(θ|x)=P(X=x|θ)


对贝叶斯定义各部分的解释如下：

+ 后验概率：即P(A|B)，是在事件B发生的条件下事件A发生的可能性的大小；
+ 先验概率：即P(A)，是根据以往经验和分析得到的某事件的概率；
+ 似然概率：即P(B|A)，似然是关于参数的函数L(θ|x)，即在参数θ给定的条件下，对于观察到的x的值的条件分布，即概率P(X=x|θ)，也就是L(θ|x)=P(X=x|θ)；
+ 边际似然概率(或者归一化因子)：即P(B)，作为上述似然概率的推广，那么边际似然概率也就是在二维随机变量θ与X的分布中，随机变量X的边际概率P(X=x)。

由于归一化因子常用希腊字母α来表示，那么公式就变为

![](/images/article/bayes7.png)

这是在二为特征空间内的贝叶斯定理，推广到高维特征空间，那么贝叶斯定理如下表示

![](/images/article/bayes8.png)

在朴素贝叶斯算法中，我们就是通过计算先验概率、似然概率与边际似然概率来估计后验概率发生的可能性，从而实现预测分类。

## 二、朴素贝叶斯算法

### 2.1 朴素贝叶斯算法的形象解释

考虑有如下训练集，其中二为特征空间中绿点有40个，红点为20个，总数为60

![](/images/article/bayes3.gif)

我们有先验概率公式

![](/images/article/bayes4.gif)

那么我们就可以根据训练集数据得到如下先验概率(根据以往经验分析得到，即根据训练集数据得到单变量的概率分布)

![](/images/article/bayes5.gif)

如果此时加入一个新的案例(如下图所示)，我们假定，该案例的类别取决于其附近的案例的类别情况，那么我们不妨以该案例为圆心，画一个包含了一定数量的绿点和红点的圆出来，如下

![](/images/article/bayes6.gif)

然后我们计算属于每个类标签的圆中的点的个数。由此我们计算出可能性：

![](/images/article/bayes7.gif)

带入数值，从而

![](/images/article/bayes8.gif)

![](/images/article/bayes9.gif)

到了这里，只需要通过贝叶斯定理将先验概率与似然概率综合起来就好了，如下

![](/images/article/bayes10.gif)

即朴素贝叶斯算法则是综合似然概率与先验概率来产生最终的后验概率，即最终的分类预测概率。上述概率并未进行归一化(标准化)操作。但是，这不会影响分类结果，因为它们的归一化常数是相同的。

## 2.2 朴素贝叶斯算法的数学定义

考虑一个数据集：

![](/images/article/bayes11.png)

以上，为了简单起见，每个向量被表示为：

![](/images/article/bayes12.png)

再定义如下目标数据集，分别对应上述数据集案例的类别：

![](/images/article/bayes13.png)

那么根据条件独立下的贝叶斯定理，就有：

![](/images/article/bayes14.png)

即通过频率计数获得边际先验概率P(y)和条件概率P(xi|y)的值，那么给定输入向量x，预测得到的类就是后验概率最大的类。

## 三、R语言中朴素贝叶斯函数及应用

### 3.1 R语言中朴素贝叶斯函数

使用e1071添加包中的函数`naiveBayes()`来实现朴素贝叶斯算法，原型如下

先创建分类器

```
m <- naiveBayes(train, class, laplace = 0)
```

+ train：训练集数据框；
+ class：训练集数据框案例对应的类别，是一个因子型向量；
+ laplace：控制拉普拉斯估计得一个数值，默认为0.

该函数返回一个朴素贝叶斯对象，能够用于预测。

进行预测

```
p <- predict(m, test, type = "class")
```

+ m：naiveBayes()函数得到的模型；
+ test：测试集数据框；
+ type：值为`class`或者`raw	`，分别对应预测的类与概率。

该函数返回一个向量，根据type参数的值决定该向量是预测类还是预测类概率。

### 3.2 R语言中朴素贝叶斯函数应用

我们使用[分而治之——决策树学习算法](http://localhost:8000/decision-tree-learning.html)一文中根据银行贷款数据来建立模型来预测贷款是否违约为例来说明朴素贝叶斯算法，只将C5.0决策树算法调整为朴素贝叶斯，其余代码不变，如下

```
> credit_train_class <- credit_train$default
> credit_test_class <- credit_test$default
> credit_train <- credit_train[-17]
> credit_test <- credit_test[-17]
> library(e1071)
> m <- naiveBayes(credit_train, credit_train_class)
> p <- predict(m, credit_test)
> prop.table(table(credit_test_class == p))

FALSE  TRUE 
 0.19  0.81 
```

评估模型效果发现，性能还是可以的。

### 3.3 调整模型性能

可以通过调整`naiveBayes()`函数中`laplace`参数的值来调整模型性能。设置`laplace`的值为1试试，如下
```
> m <- naiveBayes(credit_train, credit_train_class, laplace = 1)
> p <- predict(m, credit_test)
> prop.table(table(credit_test_class == p))

FALSE  TRUE 
  0.2   0.8 
```

模型性能稍微降低，可见之前的拟合效果还是不错的。


###### 参考文献

1. http://www.statsoft.com/Textbook/Naive-Bayes-Classifier
2. Machine Learning Algorithms / Giuseppe Bonaccorso.

全文完！