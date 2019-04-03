title: "懒惰学习——K最近邻学习算法"
date: 2019-04-02 16:26:50 +0800
update: 2019-04-02 16:26:50 +0800
author: me
cover: "-/images/knn.png"
tags:
    - Machine Learning
preview: k最近邻分类器就是把未标记的案例归类为与它们最相似的带有标记的案例所在的类，广泛适用于数据特征与目标类之间的关系众多且复杂，用其它方式难以理解，但是具有相似类的项目又非常类似的分类任务。

---

kNN(k Nearest Neighbor)是一种非参数和惰性学习算法。非参数意味着没有基础数据分布的假设。换句话说，模型结构由数据集特别是训练集确定。在大多数现实世界数据集不遵循数学理论假设的实践中，这是非常有用的。懒惰算法意味着它不需要任何训练数据点就可以生成模型。在分类器的训练阶段会使用所有的训练集数据。但也就意味着扫描数据点时间的增长和内存占用的提高。

## 一、kNN学习算法的原理

该算法是基于案例之间的相似度进行分类的，训练集负责训练一个分类器，而测试集直接将案例带入就可以通过k个最相近的训练集案例进行投票以确定最终分类结果。

首先从字面意思来解释，kNN又名K最近邻，有两层含义，分别是

+ k：测试集数据的预测类需要由与它最近的K个邻居投票表决而定，哪类邻居的票数多，该案例就属于哪一类；
+ 最近邻：与测试集数据点距离最近的训练集数据，是为该案例的最近邻。

那么综上所述，k最近邻的本质就是**计算特征空间中测试集案例关于该特征空间中训练集数据点(类)的距离最近的k个训练集数据点(邻居)的距离，测试集案例的预测类通过这k个训练集数据的投票而定，哪一类票数多，测试集案例就属于哪一类**。

### 1.1 距离的衡量

同K均值聚类一样，该算法仍然是基于距离的(也可以说是案例之间的相似度，相似度通过距离来衡量)，距离的计算常用的仍然是欧式距离。公式如下

![](/images/article/k-means1.png)

其中，x和y分别为需要计算的两个案例，xi和yi分别为两个案例相对应的第i个特征的值。这样就足够度量测试集案例同周边k个相邻训练集案例的距离了，也就能找到最近邻。

### 1.2 k的选择

事实上，没有最优的k值适合所有类型的数据集。每个数据集都有自己的要求。如果k值较小，噪声将对结果具有更高的影响而且决策边界不是很平滑。如果k值较大，那么可以较少噪声数据对模型的影响或者减少噪声导致的模型波动，从而使得决策边界更为光滑，但是也意味着昂贵的计算成本。

我们考虑两种极端的情况，如果k值为1，即测试集案例的预测类只取决于与它最相近的一个训练集数据点的类型，那么单一的近邻会使得噪声数据或者异常值过度影响案例的分类结果，一言堂是不可取的。如果k值非常大，甚至于等于训练集案例中所有观测值的数量，那么过度拟合的风险就极高，颇有一种仗势欺人的感觉。

![](/images/article/knn1.png)

因此，合理的k值应该介于这两种极端之间，并考虑训练数据中案例的数量，一种较为常见的做法是设置k值等于训练集中案例数量的平方根，但是对于训练集数据体量庞大的情况下仍然是不可取的。一般情况下k通常取3 — 10之间的某个数，

![](/images/article/knn2.png)

### 1.3 特征值标准化

在应用kNN算法之前，必不可少的操作是将特征值转化到一个合理的区间内。这种操作的合理性在于，如果某个特征具有比其他特征更大的值，那么距离的度量就会强烈地被这个庞大的值所支配，所以必须避免这种情况。

较为常用的数据标准化方法包括如下两种：

+ min-max 标准化：就是特征X的每一个值减去该特征的最小值再除以特征X的值域，该方法会将特征值映射到0 — 1范围内，公式如下

![](/images/article/knn3.png)

+ z-score 标准化：特征X的每一个值减去特征X的均值，然后再除以特征X的标准差。经过处理的数据符合标准正态分布，即均值为0，标准差为1，公式为

![](/images/article/knn4.png)

## 二、R语言中kNN函数及应用

### 2.1 R语言中kNN函数

通过使用class添加包中的`knn()`函数来应用KNN算法，原型如下

创建分类器并进行预测

```
p <- knn(train, test, cl, k)
```

+ train：一个包含数值型数据的训练集数据框；
+ test：一个包含数值型数的测试集数据框；
+ cl：包含训练集数据每一行分类的一个因子向量；
+ k：标识最近邻数目的一个整数。

该函数返回一个因子向量，该向量包含测试集数据框中每一行的预测分类。

### 2.2 R语言中KNN函数应用

以《Machine Learning With R》一书中的乳腺癌的识别为例，来说明kNN算法对数值型数据的预测分类能力，先读取数据并查看结构

```
> getwd()
[1] "C:/Users/Administrator/Documents"
> setwd("C:\\Users\\Administrator\\Desktop\\docs\\Machine-Learning-with-R-datasets-master")
> wb <- read.csv("wisc_bc_data.csv", stringsAsFactors = FALSE)
> str(wb)
'data.frame':	569 obs. of  32 variables:
 $ id                     : int  842302 842517 84300903 84348301 84358402 843786 844359 84458202 844981 84501001 ...
 $ diagnosis              : chr  "M" "M" "M" "M" ...
 $ radius_mean            : num  18 20.6 19.7 11.4 20.3 ...
 $ texture_mean           : num  10.4 17.8 21.2 20.4 14.3 ...
 $ perimeter_mean         : num  122.8 132.9 130 77.6 135.1 ...
 $ area_mean              : num  1001 1326 1203 386 1297 ...
 $ smoothness_mean        : num  0.1184 0.0847 0.1096 0.1425 0.1003 ...
 $ compactness_mean       : num  0.2776 0.0786 0.1599 0.2839 0.1328 ...
 $ concavity_mean         : num  0.3001 0.0869 0.1974 0.2414 0.198 ...
 $ concave.points_mean    : num  0.1471 0.0702 0.1279 0.1052 0.1043 ...
 $ symmetry_mean          : num  0.242 0.181 0.207 0.26 0.181 ...
 $ fractal_dimension_mean : num  0.0787 0.0567 0.06 0.0974 0.0588 ...
 $ radius_se              : num  1.095 0.543 0.746 0.496 0.757 ...
 $ texture_se             : num  0.905 0.734 0.787 1.156 0.781 ...
 $ perimeter_se           : num  8.59 3.4 4.58 3.44 5.44 ...
 $ area_se                : num  153.4 74.1 94 27.2 94.4 ...
 $ smoothness_se          : num  0.0064 0.00522 0.00615 0.00911 0.01149 ...
 $ compactness_se         : num  0.049 0.0131 0.0401 0.0746 0.0246 ...
 $ concavity_se           : num  0.0537 0.0186 0.0383 0.0566 0.0569 ...
 $ concave.points_se      : num  0.0159 0.0134 0.0206 0.0187 0.0188 ...
 $ symmetry_se            : num  0.03 0.0139 0.0225 0.0596 0.0176 ...
 $ fractal_dimension_se   : num  0.00619 0.00353 0.00457 0.00921 0.00511 ...
 $ radius_worst           : num  25.4 25 23.6 14.9 22.5 ...
 $ texture_worst          : num  17.3 23.4 25.5 26.5 16.7 ...
 $ perimeter_worst        : num  184.6 158.8 152.5 98.9 152.2 ...
 $ area_worst             : num  2019 1956 1709 568 1575 ...
 $ smoothness_worst       : num  0.162 0.124 0.144 0.21 0.137 ...
 $ compactness_worst      : num  0.666 0.187 0.424 0.866 0.205 ...
 $ concavity_worst        : num  0.712 0.242 0.45 0.687 0.4 ...
 $ concave.points_worst   : num  0.265 0.186 0.243 0.258 0.163 ...
 $ symmetry_worst         : num  0.46 0.275 0.361 0.664 0.236 ...
 $ fractal_dimension_worst: num  0.1189 0.089 0.0876 0.173 0.0768 ...
```

id列并没有实际意义，应该剔除

```
> wb <- wb[-1]
```

预测类特征为diagnosis，应该是一个因子型向量，但是现在明显不是，所以应该转化一下

```
> str(wb$diagnosis)
 chr [1:569] "M" "M" "M" "M" "M" "M" "M" "M" "M" "M" "M" "M" "M" "M" "M" "M" ...
> wb$diagnosis <- factor(wb$diagnosis, levels = c("B", "M"), labels = c("Benign", "Malignant"))
> str(wb$diagnosis)
 Factor w/ 2 levels "Benign","Malignant": 2 2 2 2 2 2 2 2 2 2 ...
```

从上述数据结构也可以看出，某些特征的值非常大，而某些特征的值又很小，所以必须进行标准化操作，如下

```
> normalize <- function(x) {
+   return ((x - min(x)) / (max(x) - min(x)))
+ }
> wb.normalize <- as.data.frame(lapply(wb[2 : 31], normalize))
```

然后分别构造训练集与测试集，如下

```
> wb.normalize.train <- wb.normalize[1 : 469, ]
> wb.normalize.test <- wb.normalize[470 : 569, ]
```

由于我们标准化的数据剔除了目标特征，在建模阶段需要训练集的目标特征，而在测试集验证中需要测试集的目标特征，所以我们从标准化之前的数据框中提取该列向量，如下

```
> wb.normalize.train.labels <- wb[1 : 469, 1]
> wb.normalize.test.labels <- wb[470 : 569, 1]
```

现在就可以创建分类器并预测了，如下

```
> p <- knn(train = wb.normalize.train, test = wb.normalize.test, cl = wb.normalize.train.labels, k = 3)
```

### 2.3 评估模型性能

```
> prop.table(table(p == wb.normalize.test.labels))

FALSE  TRUE 
 0.06  0.94 
```

可见正确率为94%，还是相当不错的，也可以通过双向交叉表来查看，如下

```
> table(wb.normalize.test.labels, p)
                        p
wb.normalize.test.labels Benign Malignant
               Benign        72         5
               Malignant      1        22
```

或者

```
> library(gmodels)
> CrossTable(x = wb.normalize.test.labels, y = p, prop.chisq = FALSE)

 
   Cell Contents
|-------------------------|
|                       N |
|           N / Row Total |
|           N / Col Total |
|         N / Table Total |
|-------------------------|

 
Total Observations in Table:  100 

 
                         | p 
wb.normalize.test.labels |    Benign | Malignant | Row Total | 
-------------------------|-----------|-----------|-----------|
                  Benign |        72 |         5 |        77 | 
                         |     0.935 |     0.065 |     0.770 | 
                         |     0.986 |     0.185 |           | 
                         |     0.720 |     0.050 |           | 
-------------------------|-----------|-----------|-----------|
               Malignant |         1 |        22 |        23 | 
                         |     0.043 |     0.957 |     0.230 | 
                         |     0.014 |     0.815 |           | 
                         |     0.010 |     0.220 |           | 
-------------------------|-----------|-----------|-----------|
            Column Total |        73 |        27 |       100 | 
                         |     0.730 |     0.270 |           | 
-------------------------|-----------|-----------|-----------|
```

可见，共有1个原本是Benign的却被预测为Malignant，而有5个原本是Malignant的，却被预测为Benign。

### 2.4 提高模型的性能

提高模型的性能可以通过调整k值来实现，试着调大k值

```
> p <- knn(train = wb.normalize.train, test = wb.normalize.test, cl = wb.normalize.train.labels, k = 5)
> prop.table(table(p == wb.normalize.test.labels))

FALSE  TRUE 
 0.03  0.97 
> p <- knn(train = wb.normalize.train, test = wb.normalize.test, cl = wb.normalize.train.labels, k = 7)
> prop.table(table(p == wb.normalize.test.labels))

FALSE  TRUE 
 0.02  0.98 
> p <- knn(train = wb.normalize.train, test = wb.normalize.test, cl = wb.normalize.train.labels, k = 9)
> prop.table(table(p == wb.normalize.test.labels))

FALSE  TRUE 
 0.02  0.98 
> p <- knn(train = wb.normalize.train, test = wb.normalize.test, cl = wb.normalize.train.labels, k = 11)
> prop.table(table(p == wb.normalize.test.labels))

FALSE  TRUE 
 0.02  0.98 
```

可见，分类器表现不错，再试试两个极端的k值？如下

```
> p <- knn(train = wb.normalize.train, test = wb.normalize.test, cl = wb.normalize.train.labels, k = 1)
> prop.table(table(p == wb.normalize.test.labels))

FALSE  TRUE 
 0.07  0.93 
> p <- knn(train = wb.normalize.train, test = wb.normalize.test, cl = wb.normalize.train.labels, k = 469)
> prop.table(table(p == wb.normalize.test.labels))

FALSE  TRUE 
 0.23  0.77 
```

明显不如介于两个极端之间的K值拟合效果好。

全文完！