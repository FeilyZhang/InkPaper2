title: "物以类聚——K-Means Clustering学习算法"
date: 2019-03-31 13:43:14 +0800
update: 2019-03-31 13:43:14 +0800
author: me
cover: "-/images/k-means.png"
tags:
    - Machine Learning
preview: 聚类学习被认为是典型的无监督学习算法，训练集数据没有目标特征。聚类的目标就是探索数据集中蕴藏的内在结构，并将相似或同质的群组数据聚集在一起。

---

与有监督学习算法不同，无监督学习算法的训练集数据并没有目标特征，我们不知道有哪些类别以及有几个类别，只能通过算法探索将相似度较高或者同质性强的样本聚类在一起，最终实现对无标签案例的分类任务。

## 一、K-Means Clustering 算法

K均值(K-Means Clustering)是一种数据聚类算法，可用于无监督的机器学习。它能够将类似的非标记数据组合到预定数量的簇中。

### 1.1 K均值聚类与kNN最近邻的对比

K均值聚类在某些程度上与kNN最近邻有监督学习算法有相似之处，如下：

1. 都是将相似度高或者同质性强的样本对象集聚在一起；
2. 都包含k个簇，每个簇代表一类样本；
3. 都是基于距离(特别是欧式距离)的分类或聚类算法。

二者也有区别，如下：

1. K均值属于无监督学习算法，kNN属于有监督学习算法；
2. 由于K均值无监督学习算法的特征，数据集并没有目标特征，全靠算法探索；而kNN有监督学习算法则拥有目标特征；
3. kNN被认为是懒惰的、机械的，模型性能或泛化能力高度依赖于训练集数据的泛性，如果训练集数据极具概括性，那么模型的泛化能力很强而且训练时间短，反之则模型泛化能力差；而K均值则由于模型的迭代修正使得预测结果更加稳健。

### 1.2 K均值聚类的原理

在训练阶段，**K均值聚类的目标是将n个训练集样本根据其特征集，将样本分配给与某簇相似的或者与某簇质心距离最近的簇中，而与当前簇距离较远的样本归入另外的簇中**。其中，簇是一个对象的集合，**簇中的对象(样本)相似度极高，簇间差异化极大**。

该算法本质上包含以下步骤：

1. 首先初始化簇质心，一种办法是从训练集数据中随机选择k个案例(这里的随机并不意味着训练结果的不精确，因为之后会通过迭代重新选择质心优化模型)；
2. 计算训练集各案例到各簇质心的距离，然后将该案例归入距离最小的簇中；
3. 训练集案例分配完毕后，重新计算各簇的质心(通过计算训练集特征空间中各特征的均值构造质心)；重复第2步和第3步，直至迭代操作不会再提升类优度为止。

### 1.3 K均值通过距离实现类的更新与分配

通常情况下，K均值使用欧式距离(欧式距离计算两点之间的距离，即两点之间的连线)来计算两个案例之间的距离，公式定义如下：

![](/images/article/k-means1.png)

其中，x和y分别代表两个案例，xi与yi分别代表两个案例的第i个特征值。通过使用该距离函数，就可以计算两个案例之间的距离。在K-Means中，我们通过计算某案例与簇质心之间的欧式距离，然后将该案例分配给距离最近的簇。

但并不意味着K-Means只能使用欧式距离来度量案例之间的距离，其余距离公式如下

+ 曼哈顿距离：用以标明两个点在标准坐标系上的绝对轴距总和, 又名“城市街区距离”，因为城市两点之间的实际距离并不是两点之间的直线距离(因为有建筑物等阻挡)，只能是两点在标准坐标系上的绝对距离之和。

![](/images/article/k-means2.png)

+ 切比雪夫距离：是向量空间中的一种度量，二个点之间的距离定义为其各坐标数值差绝对值的最大值。

![](/images/article/k-means4.png)

+ 闵可夫斯基距离：两个n维变量a(a1, a2, ..., an)与 b(b1, b2, ...,   bn)间的闵可夫斯基距离定义(其中p是一个可变参数，特别的，当p=1时，就是曼哈顿距离；当p=2时，就是欧氏距离；当p→∞时，就是切比雪夫距离。)为

![](/images/article/k-means3.png)

### 1.4 聚类中K的选择

影响K均值聚类模型的最重要的因素就是K的选择，过大的K利于提升簇内的同质性与簇间的异质性，但是会有过度拟合的风险，过小的K则无法识别类之间的差异。因此，K的选择应该是合适就好，一般有以下方法：

+ 如果具备真实分组先验知识，那么我们就会提前知道我们想分为几类，那么K就是几，当然这是最理想的；
+ 如果不具备先验知识，那么一个经验性的规则就是令`K = (n / 2) ^ (1 / 2)`，但是对于大样本而言，这无疑仍然会导一个较大的K值；
+ 一种被称为“肘部法”的技术用于衡量关于K值选择的合适的度。一般情况下，**随着K值得增大，簇内部的同质性强增强，同样的，簇间的异质性也会增强；但是随着k值得继续增大，簇内部的同质性仍然在增强，但是簇间的异质性却在降低，因为过度拟合**。**理想的情况是，当簇内的同质性与簇间的异质性达到最大时，就应该立即停止K的增大**，这时，K就是最合适的K。

## 二、K-Means Clustering 的数学定义

### 2.1 数学定义
K均值聚类在实际应用过程中，是用局部最优解来逐步实现全局最优的。即每一次迭代中根据簇内均值计算簇质心，然后将所有训练集样本重新分配，确保每次迭代中每一个样本都是满足“到达某簇质心的距离最小”这一条件的。数学定义如下

各簇的定义为：

![](/images/article/k-means8.png)

可见，分为了c个簇，所有的训练集样本将会被分配到这c个簇当中；

初始阶段，簇质心随机选择c个。而更新迭代阶段，簇质心的定义(更新规则)为：

![](/images/article/k-means6.png)

即对当前簇中各特征求和再除以簇内样本总数，即求均值，然后就会得到簇质心的坐标。

而在整个模型训练中，对样本分配簇被定义为：

![](/images/article/k-means7.png)

其中xi为训练集中某一个样本的特征向量，uy为簇质心的特征向量，为了与绝对值区分，`||xi-uy||`代表两点之间的距离(可以理解为具体距离公式的统称)，那么整个式子的含义是**计算该样本与各个簇心的距离，并为该样本赋予距离最近的簇心类别，即将该样本纳入该簇当中**。上式也就是为达到全局最优解而拆分的局部最优解。

### 2.2 算法的规范描述

基于上述数学定义，那么算法的规范描述为：

1. 给各个簇中心u1, u2, ..., uc以适当的初始值; 
2. 更新样本x1, x2, ..., xn对应的聚类标签y1, y2, ..., yn：
![](/images/article/k-means7.png)
3. 更新各个簇中心u1, u2, ..., uc：
![](/images/article/k-means6.png)
4. 直到聚类标签达到收敛精度为止，否则重复上述2，3步的计算。

## 三、R语言中 K-Means Clustering 函数及应用

### 3.1 K-Means Clustering 函数
使用stats添加包中的`kmeans()`函数来实现K-Means Clustering，原型如下

首先建立模型

```
clusters <- kmeans(mydata, k)
```

+ mydata：数据框，包含需要聚类的实例；
+ k：聚类的个数；

该函数返回一个含有K均值聚类结果的对象。

然后检查聚类结果

+ `clusters$cluster`是`kmeans()`函数所给出的类成员变量；
+ `clusters$centers`是含有每个类组合和每一个特征的均值的一个矩阵；
+ `clusters$size`给出每一个类中的实例个数

### 3.2  K-Means Clustering 函数应用

本例使用K均值聚类探索青少年市场细分，共包含30 000名青少年随记案例数据集，由于是无监督机器学习，那么我们不需要拆分数据集为训练集与测试集，所有的数据均为训练集或者样本，首先读取数据

```
> getwd()
[1] "C:/Users/Administrator/Documents"
> setwd("C:\\Users\\Administrator\\Desktop\\docs\\Machine-Learning-with-R-datasets-master")
> tee <- read.csv("snsdata.csv")
> str(tee)
'data.frame':	30000 obs. of  40 variables:
 $ gradyear    : int  2006 2006 2006 2006 2006 2006 2006 2006 2006 2006 ...
 $ gender      : Factor w/ 2 levels "F","M": 2 1 2 1 NA 1 1 2 1 1 ...
 $ age         : num  19 18.8 18.3 18.9 19 ...
 $ friends     : int  7 0 69 0 10 142 72 17 52 39 ...
 $ basketball  : int  0 0 0 0 0 0 0 0 0 0 ...
 $ football    : int  0 1 1 0 0 0 0 0 0 0 ...
 $ soccer      : int  0 0 0 0 0 0 0 0 0 0 ...
 $ softball    : int  0 0 0 0 0 0 0 1 0 0 ...
 $ volleyball  : int  0 0 0 0 0 0 0 0 0 0 ...
 $ swimming    : int  0 0 0 0 0 0 0 0 0 0 ...
 $ cheerleading: int  0 0 0 0 0 0 0 0 0 0 ...
 $ baseball    : int  0 0 0 0 0 0 0 0 0 0 ...
 $ tennis      : int  0 0 0 0 0 0 0 0 0 0 ...
 $ sports      : int  0 0 0 0 0 0 0 0 0 0 ...
 $ cute        : int  0 1 0 1 0 0 0 0 0 1 ...
 $ sex         : int  0 0 0 0 1 1 0 2 0 0 ...
 $ sexy        : int  0 0 0 0 0 0 0 1 0 0 ...
 $ hot         : int  0 0 0 0 0 0 0 0 0 1 ...
 $ kissed      : int  0 0 0 0 5 0 0 0 0 0 ...
 $ dance       : int  1 0 0 0 1 0 0 0 0 0 ...
 $ band        : int  0 0 2 0 1 0 1 0 0 0 ...
 $ marching    : int  0 0 0 0 0 1 1 0 0 0 ...
 $ music       : int  0 2 1 0 3 2 0 1 0 1 ...
 $ rock        : int  0 2 0 1 0 0 0 1 0 1 ...
 $ god         : int  0 1 0 0 1 0 0 0 0 6 ...
 $ church      : int  0 0 0 0 0 0 0 0 0 0 ...
 $ jesus       : int  0 0 0 0 0 0 0 0 0 2 ...
 $ bible       : int  0 0 0 0 0 0 0 0 0 0 ...
 $ hair        : int  0 6 0 0 1 0 0 0 0 1 ...
 $ dress       : int  0 4 0 0 0 1 0 0 0 0 ...
 $ blonde      : int  0 0 0 0 0 0 0 0 0 0 ...
 $ mall        : int  0 1 0 0 0 0 2 0 0 0 ...
 $ shopping    : int  0 0 0 0 2 1 0 0 0 1 ...
 $ clothes     : int  0 0 0 0 0 0 0 0 0 0 ...
 $ hollister   : int  0 0 0 0 0 0 2 0 0 0 ...
 $ abercrombie : int  0 0 0 0 0 0 0 0 0 0 ...
 $ die         : int  0 0 0 0 0 0 0 0 0 0 ...
 $ death       : int  0 0 1 0 0 0 0 0 0 0 ...
 $ drunk       : int  0 0 0 0 1 1 0 0 0 0 ...
 $ drugs       : int  0 0 0 0 1 0 0 0 0 0 ...
```

观察到`gender`列存在缺失值，不妨验证一下

```
> table(tee$gender)
    F     M 
22054  5222 
> table(tee$gender, useNA = "ifany")
    F     M  <NA> 
22054  5222  2724
```

可见在`gender`列共有2724条个缺失值，其它列有缺失值吗，不妨再看看

```
> summary(tee)
    gradyear     gender           age             friends         basketball     
 Min.   :2006   F   :22054   Min.   :  3.086   Min.   :  0.00   Min.   : 0.0000  
 1st Qu.:2007   M   : 5222   1st Qu.: 16.312   1st Qu.:  3.00   1st Qu.: 0.0000  
 Median :2008   NA's: 2724   Median : 17.287   Median : 20.00   Median : 0.0000  
 Mean   :2008                Mean   : 17.994   Mean   : 30.18   Mean   : 0.2673  
 3rd Qu.:2008                3rd Qu.: 18.259   3rd Qu.: 44.00   3rd Qu.: 0.0000  
 Max.   :2009                Max.   :106.927   Max.   :830.00   Max.   :24.0000  
                             NA's   :5086                                        
    football           soccer           softball         volleyball     
 Min.   : 0.0000   Min.   : 0.0000   Min.   : 0.0000   Min.   : 0.0000  
 1st Qu.: 0.0000   1st Qu.: 0.0000   1st Qu.: 0.0000   1st Qu.: 0.0000  
 Median : 0.0000   Median : 0.0000   Median : 0.0000   Median : 0.0000  
 Mean   : 0.2523   Mean   : 0.2228   Mean   : 0.1612   Mean   : 0.1431  
 3rd Qu.: 0.0000   3rd Qu.: 0.0000   3rd Qu.: 0.0000   3rd Qu.: 0.0000  
 Max.   :15.0000   Max.   :27.0000   Max.   :17.0000   Max.   :14.0000  
                                                                        
    swimming        cheerleading       baseball           tennis        
 Min.   : 0.0000   Min.   :0.0000   Min.   : 0.0000   Min.   : 0.00000  
 1st Qu.: 0.0000   1st Qu.:0.0000   1st Qu.: 0.0000   1st Qu.: 0.00000  
 Median : 0.0000   Median :0.0000   Median : 0.0000   Median : 0.00000  
 Mean   : 0.1344   Mean   :0.1066   Mean   : 0.1049   Mean   : 0.08733  
 3rd Qu.: 0.0000   3rd Qu.:0.0000   3rd Qu.: 0.0000   3rd Qu.: 0.00000  
 Max.   :31.0000   Max.   :9.0000   Max.   :16.0000   Max.   :15.00000  
                                                                        
     sports           cute              sex                sexy        
 Min.   : 0.00   Min.   : 0.0000   Min.   :  0.0000   Min.   : 0.0000  
 1st Qu.: 0.00   1st Qu.: 0.0000   1st Qu.:  0.0000   1st Qu.: 0.0000  
 Median : 0.00   Median : 0.0000   Median :  0.0000   Median : 0.0000  
 Mean   : 0.14   Mean   : 0.3229   Mean   :  0.2094   Mean   : 0.1412  
 3rd Qu.: 0.00   3rd Qu.: 0.0000   3rd Qu.:  0.0000   3rd Qu.: 0.0000  
 Max.   :12.00   Max.   :18.0000   Max.   :114.0000   Max.   :18.0000  
                                                                       
      hot              kissed            dance              band        
 Min.   : 0.0000   Min.   : 0.0000   Min.   : 0.0000   Min.   : 0.0000  
 1st Qu.: 0.0000   1st Qu.: 0.0000   1st Qu.: 0.0000   1st Qu.: 0.0000  
 Median : 0.0000   Median : 0.0000   Median : 0.0000   Median : 0.0000  
 Mean   : 0.1266   Mean   : 0.1032   Mean   : 0.4252   Mean   : 0.2996  
 3rd Qu.: 0.0000   3rd Qu.: 0.0000   3rd Qu.: 0.0000   3rd Qu.: 0.0000  
 Max.   :10.0000   Max.   :26.0000   Max.   :30.0000   Max.   :66.0000  
                                                                        
    marching           music              rock              god         
 Min.   : 0.0000   Min.   : 0.0000   Min.   : 0.0000   Min.   : 0.0000  
 1st Qu.: 0.0000   1st Qu.: 0.0000   1st Qu.: 0.0000   1st Qu.: 0.0000  
 Median : 0.0000   Median : 0.0000   Median : 0.0000   Median : 0.0000  
 Mean   : 0.0406   Mean   : 0.7378   Mean   : 0.2433   Mean   : 0.4653  
 3rd Qu.: 0.0000   3rd Qu.: 1.0000   3rd Qu.: 0.0000   3rd Qu.: 1.0000  
 Max.   :11.0000   Max.   :64.0000   Max.   :21.0000   Max.   :79.0000  
                                                                        
     church            jesus             bible               hair        
 Min.   : 0.0000   Min.   : 0.0000   Min.   : 0.00000   Min.   : 0.0000  
 1st Qu.: 0.0000   1st Qu.: 0.0000   1st Qu.: 0.00000   1st Qu.: 0.0000  
 Median : 0.0000   Median : 0.0000   Median : 0.00000   Median : 0.0000  
 Mean   : 0.2482   Mean   : 0.1121   Mean   : 0.02133   Mean   : 0.4226  
 3rd Qu.: 0.0000   3rd Qu.: 0.0000   3rd Qu.: 0.00000   3rd Qu.: 0.0000  
 Max.   :44.0000   Max.   :30.0000   Max.   :11.00000   Max.   :37.0000  
                                                                         
     dress           blonde              mall            shopping     
 Min.   :0.000   Min.   :  0.0000   Min.   : 0.0000   Min.   : 0.000  
 1st Qu.:0.000   1st Qu.:  0.0000   1st Qu.: 0.0000   1st Qu.: 0.000  
 Median :0.000   Median :  0.0000   Median : 0.0000   Median : 0.000  
 Mean   :0.111   Mean   :  0.0989   Mean   : 0.2574   Mean   : 0.353  
 3rd Qu.:0.000   3rd Qu.:  0.0000   3rd Qu.: 0.0000   3rd Qu.: 1.000  
 Max.   :9.000   Max.   :327.0000   Max.   :12.0000   Max.   :11.000  
                                                                      
    clothes         hollister        abercrombie           die         
 Min.   :0.0000   Min.   :0.00000   Min.   :0.00000   Min.   : 0.0000  
 1st Qu.:0.0000   1st Qu.:0.00000   1st Qu.:0.00000   1st Qu.: 0.0000  
 Median :0.0000   Median :0.00000   Median :0.00000   Median : 0.0000  
 Mean   :0.1485   Mean   :0.06987   Mean   :0.05117   Mean   : 0.1841  
 3rd Qu.:0.0000   3rd Qu.:0.00000   3rd Qu.:0.00000   3rd Qu.: 0.0000  
 Max.   :8.0000   Max.   :9.00000   Max.   :8.00000   Max.   :22.0000  
                                                                       
     death             drunk             drugs         
 Min.   : 0.0000   Min.   :0.00000   Min.   : 0.00000  
 1st Qu.: 0.0000   1st Qu.:0.00000   1st Qu.: 0.00000  
 Median : 0.0000   Median :0.00000   Median : 0.00000  
 Mean   : 0.1142   Mean   :0.08797   Mean   : 0.06043  
 3rd Qu.: 0.0000   3rd Qu.:0.00000   3rd Qu.: 0.00000  
 Max.   :14.0000   Max.   :8.00000   Max.   :16.00000  
 ```
 
发现只有age列和gender列存在缺失值，对待缺失值我们要么查补要么直接删除记录，我们这里使用查补的办法。
 
再观察上述基本统计量，我们发现年龄的最小值为3.086，最大值为106.927，这明显是错误的，我们假定青少年的年龄区间是`[13, 20)`，那么我们定义超出该区间的所有数据为异常值，对于异常值，我们可以将其视作缺失值处理，也可以不处理，这里明显需要处理。
 
首先，将不符合年龄区间的值统一设置为缺失值，如下
 
```
> tee$age <- ifelse(tee$age >= 13 & tee$age < 20, tee$age, NA)
> summary(tee$age)
   Min. 1st Qu.  Median    Mean 3rd Qu.    Max.    NA's 
  13.03   16.30   17.27   17.25   18.22   20.00    5523 
```

接下来需要将年龄的缺失值进行查补，这里使用均值插补法，考虑到毕业年份`gradyear`和年龄`age`之间的关系，我们要计算多次均值来查补，使用`ave()`函数来计算`age`列关于`gradyear`列中各水平的均值，如下

```
> ave_age <- ave(tee$age, tee$gradyear, FUN = function(x) mean(x, na.rm = TRUE))
> length(ave_age)
[1] 30000
```

参数`ave_age`包含数据框中以`gradyear`列为基准的`age`列对应的值。然后进行查补

```
> tee$age <- ifelse(is.na(tee$age), ave_age, tee$age)
```

如果`tee$age`满足条件`is.na(tee$age)`那么使用该`age`对应的根据`gradyear`得到的均值来查补，否则就原封不动(因为真实的值比虚拟的均值更能说明问题)，再看一下查补结果

```
> summary(tee$age)
   Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
  13.03   16.28   17.24   17.24   18.21   20.00 
```

以上是对`age`的处理，而对`gender`的处理我们通过创建虚拟变量来拆分gender语义，因为性别我们无法计算均值，首先我们创建虚拟变量`female`将所有的女性设置为变量1，其余均为0，如下

```
> tee$female <- ifelse(tee$gender == "F" & !is.na(tee$gender), 1, 0)
> table(tee$female)
    0     1 
 7946 22054 
```

然后再创建虚拟变量`no_gender`将`gender`中所有的缺失值设置为1，其余为0如下

```
> tee$no_gender <- ifelse(is.na(tee$gender), 1, 0)
> table(tee$no_gender)

    0     1 
27276  2724 
```

如下，我们通过比较虚拟变量`female`与`no_gender`和变量`gender`发现，虚拟变量值为1的数目与变量`gender`中值为`F`和`NA`的数目是一致的。

```
> table(tee$gender, useNA = "ifany")

    F     M  <NA> 
22054  5222  2724 
> table(tee$female, useNA = "ifany")

    0     1 
 7946 22054 
> table(tee$no_gender, useNA = "ifany")

    0     1 
27276  2724 
```