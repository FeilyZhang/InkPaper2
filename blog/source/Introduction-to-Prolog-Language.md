title: "Prolog 语言入门"
date: 2019-02-02 17:09:05 +0800
update: 2019-02-02 17:09:05 +0800
author: me
cover: "-/images/swipl.png"
tags:
    - Prolog
preview: 今天闲来无事，就上网搜了搜有哪些小众冷门但却实用的编程语言，毫无疑问，Prolog脱颖而出。平时总是在写结构化的逻辑代码，似乎只要知道顺序、选择、循环三种结构，再加上常用的算法和数据结构就能解决绝大多数问题，不免觉得无趣。所以对这门以逻辑标榜的编程语言甚是钟情。

---

## 一、写在前面

今天闲来无事，就上网搜了搜有哪些小众冷门但却实用的编程语言，毫无疑问，Prolog脱颖而出。平时总是在写结构化的逻辑代码，似乎只要知道顺序、选择、循环三种结构，再加上常用的算法和数据结构就能解决绝大多数问题，不免觉得无趣。所以对这门以逻辑标榜的编程语言甚是钟情。

国内网上关于Prolog的资料和文档寥寥无几，在[阮一峰老师的博客](http://www.ruanyifeng.com/blog/2019/01/prolog.html?hmsr=toutiao.io&utm_medium=toutiao.io&utm_source=toutiao.io)中倒是发现一篇文章足以入门Prolog，甚是开心，本文结合阮一峰老师的文章和原文出处[Solving murder with Prolog](https://xmonader.github.io/prolog/2018/12/21/solving-murder-prolog.html)，再加上本人实践，综合而成，感谢两位的无私奉献。

## 二、Prolog运行环境SWI-Prolog的安装

在官网下载相应操作系统的二进制安装包，然后安装，一定要将安装路径加入环境变量，一般在安装的时候会提示，勾选即可。

在命令行键入`swipl`命令即可进入Prolog的shell

```
C:\Users\Administrator>swipl
Welcome to SWI-Prolog (threaded, 64 bits, version 8.0.1)
SWI-Prolog comes with ABSOLUTELY NO WARRANTY. This is free software.
Please run ?- license. for legal details.

For online help and background, visit http://www.swi-prolog.org
For built-in help, use ?- help(Topic). or ?- apropos(Word).

1 ?- 
```

这就进入了Prolog的运行环境，键入以下命令退出shell(别忘了最后的.)

```
halt.
```

Prolog运行环境和源文件下每条语句是以`.`符号结束的，相当于Java中`;`符号的作用。

Prolog标准输出函数为`write()`. hello, world输出如下

```
1 ?- write('hello, world').
hello, world
true.
```

其中`true.`为返回结果，代表加载成功！

## 三、基本语法

### 3.1 常量和变量

常量是以小写字母开头，变量则是以大写字母开头，示例如下

```
3 ?- write(name).
name
true.

4 ?- write(Name).
_4042
true.
```

name是常量，那么会直接输出自身，Name是变量，那么输出的就是该变量的值。

### 3.2 关系和属性

关系肯定是两个事物时间的关系，方向从左到右，如下

```
friend(jack, peter).
```

意为jack的朋友是peter，即jack与peter是朋友关系，由于方向是从左到右，那么peter的朋友不一定是jack，如果我们想表达jack与peter互为朋友，那么如下表示

```
friend(jack, peter).
friend(peter, jack).
```

属性是一个事物所具有的，比如jack是男的，男的就是peter的一个属性，如下表示

```
male(jack)
```

简言之，括号里面有两个参数，那么指的就是关系，有一个参数，那么指的就是属性。

### 3.3 规则以及建立在属性、关系和规则上面程序的提问

规则是推理的依据，即如何从一个论断得到另一个论断。

我们定义一条规则：**所有的曾经的喜欢关系是相互的，即一方曾经喜欢过另一方，那么另一方也曾经喜欢过一方，即一方曾经也被另一方喜欢过**

首先我们需要定义关系，其次再利用规则，我们暂时认为关系的定义必须是具体的，即只能用小写的常量，而规则的定义必须是笼统的，即规则用大写的变量

```
love(zhangsan, lisi).
stillLove(B, A) :- love(A,B).
```

将上述代码保存在一个名为love.pl的文件中，然后用swipl-win.exe打开，显示如下

```
Welcome to SWI-Prolog (threaded, 64 bits, version 8.0.1)
SWI-Prolog comes with ABSOLUTELY NO WARRANTY. This is free software.
Please run ?- license. for legal details.

For online help and background, visit http://www.swi-prolog.org
For built-in help, use ?- help(Topic). or ?- apropos(Word).

?-
```

然后我们可以根据规则来进行提问。

我们在源文件中描述了具体的人物关系是张三爱过李四，我们想根据规则得出推论：如果A爱过B，那么B仍然爱过A。有三种提问方式，如下

第一种，我们直接使用双变量进行查询，含义为谁曾经还喜欢过谁，如下

```
?- stillLove(Who1, Who2).
Who1 = lisi,
Who2 = zhangsan.
```

我们设置了两个变量Who1和Who2，进行提问，结果显示Who1 = lisi, Who2 = zhangsan。这表示如果从zhangsan和lisi的关系出发，根据我们定义的规则，可以得出lisi曾经爱过zhangsan。

第二种，左单变量进行查询，含义为lisi曾经还喜欢过谁，如下

```
?- stillLove(lisi, Who).
Who = zhangsan.
```

仍然可以得出张三曾经喜欢过李四

第三种，右单变量查询，含义为zhangsan曾经被谁喜欢过，如下

```
?- stillLove(Who, zhangsan).
Who = lisi.
```

仍然可以得出张三曾经喜欢过李四

我们再定义一个规则：**所有的曾经的喜欢关系是相互的，即一方曾经喜欢过另一方，那么另一方也曾经喜欢过一方，即一方曾经也被另一方喜欢过，且双方必须是异性**

那么我们需要修改一下源程序,如下

```
love(zhangsan, lisi).
male(zhangsan).
female(lisi).
stillLove(B, A) :- love(A,B), male(A), female(B).
```

我们通过定义属性的形式具体的指出了zhangsan是男性，lisi是女性；并且通过笼统的在规则中增加定义A是男性，B是女性，来达到我们的目的。可以看出表示"且"这一概念的符号是英文逗号。同样的三种提问方式，得到的结果和上面一致。如果我们如下提问

同样的三种提问方式，得到的结果和上面一致。如果我们如下提问

```
?- stillLove(Who1, lisi).
false.
```

那么得出的结果是false，即zhangsan从未喜欢过lisi。很扎心吧，因为我们定义的A是男性，但是我们给的A是女性lisi，那么势必会推理出false.

**规则中还有一个符号是` \+ `,表示"不是"的意思**，我们再次改造一下上面的程序，如下

```
love(zhangsan, lisi).
love(zhangsan,wangwu).
male(zhangsan).
male(wangwu).
female(lisi).
stillLove(B, A) :- love(A,B), \+female(B).
```

在上述程序中我们假定男性张三爱过女性李四、男性张三爱过男性王五，如果根据我们上述第一个规则那么我们可以得出女性李四也喜欢过男性张三、男性王五也喜欢过男性张三；如果根据第二条规则，那么我们得出的结论是只有女性李四喜欢过张三，王五被排除在外了，为什么？因为我们的且条件是B是女性，但是王五是男性。王五很无辜，如果我们想把王五纳入把李四排除，那么我们需要修改一下条件，就是第三个规则(添加了` \+ `符号的规则)，即让B不为女性，即男性。演示为

```
?- stillLove(Who1, Who2).
Who1 = wangwu,
Who2 = zhangsan.
```
### 四、总结

**总体而言，Prolog的编程逻辑为：通过关系和属性进行事实描述 -> 添加推论的规则约束 -> shell提问得出结论**

而shell中可以使用单变量/双变量进行查询追问。