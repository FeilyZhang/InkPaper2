title: "深入理解Prolog的运行机制"
date: 2019-02-02 19:36:06 +0800
update: 2019-02-02 19:36:06 +0800
author: me
cover: "-/images/swipl.png"
tags:
    - Prolog
preview: 前面的文章只是对Prolog一些直观上的简单应用，有必要深入了解一下这门语言的运行机制。

---

前面的文章只是对Prolog一些直观上的简单应用，有必要深入了解一下这门语言的运行机制。

我们从以前的例子说起,对其进行简单改造

```
love(zhangsan, lisi).
love(zhangsan,wangwu).
male(zhangsan).
male(wangwu).
female(lisi).
stillLove(B, A) :- love(A,B), male(A), female(B).
```

将其保存为love1.pl,执行这段代码，我们可以得到下面的结论

```
?- stillLove(Who1, Who2).
Who1 = lisi,
Who2 = zhangsan ;
false.
```

那么Prolog到底是怎么运行的呢？

我们前面说过，结论stillLove(B, A)也可以看做事实，特殊之处在于该事实是由原先的事实推论出来的，也就是说应该从:-符号的右边开始读，然后得到左边的事实结论。

stillLove(Who1, Who2)相当于提问谁和谁满足这样的事实。那么既然结论事实是由符号:-右边的推论得出的，该stillLove(Who1, Who2)表达式就会`传递`到love(A,B), male(A), female(B)上面进行提问。具体操作过程如下

+ love(A, B)就回去匹配原先的事实，看`谁和谁符合love关系`，那么就会得到如下结论：
	- A = zhangsan;
	- B = {lisi, wangwu};
+ male(A)也会去匹配原先的事实，看`谁的属性是male`，那么会得到如下结论：
	- A = {zhangsan, wangwu}；
+ female(B)也会去匹配原先的事实，看`谁的属性是female`，那么会得到如下结论：
	- B = lisi

好了，到此为止符号:-右边的变量A,B所代表的常量已经找到了，那么接下来只需要综合一下就好了，即关系和相对应的属性取交集，如下

+ 关系love中的A的值与属性male中的A的值的交集是zhangsan，那么现在就找到了A是zhangsan;
+ 关系love中的B的值与属性female中的B的值的交集是lisi，那么现在也找到了B是lisi。

现在综合上述结论，即A = zhangsan, B = lisi，这就是符号:-右边的规则约束原有事实得出的结论，由于stillLove(Who1, Who2)是将查询传递到符号:-右边的规则约束上的，那么自然该结论也就是stillLove(Who1, Who2)的结论。

需要注意的是，这里还会有一个参数对应关系，即源程序中stillLove表达式中B对应的是shell中的Who1，A对应的是Who2，那么现在得出的最终结论就是

+ Who1 = lisi,
+ Who2 = zhangsan;