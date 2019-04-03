title: "Erlang程序设计(一)"
date: 2019-04-02 18:42:05 +0800
update: 2019-04-02 19:52:05 +0800
author: me
cover: "-/images/E-LearningLogo.png"
tags:
    - Erlang
preview: 常量与变量、常用数据类型。

---

## 一、常量与变量

Erlang中所有的变量名必须以大写字母开头，且变量属于**一次性赋值变量**，即只能被赋值一次，如果试图在变量被设置后再改变它的值就会得到一个错误。

在Erlang里，变量不过是对某个值的引用，Erlang的实现方式是用指针指向一个包含值得内存区，这个值不能被修改。这样做的好处就是不存在共享内存，也就不存在锁，所以让并发实现的更简单。

```
C:\Users\Administrator>erl
Eshell V10.0.1  (abort with ^G)
1> X = 1.
1
2> X.
1
3> X = 2.
** exception error: no match of right hand side value 2
```

而Erlang中的常量(或者叫原子)是直接以小写字母开头的，可以直接使用，无需定义

```
4> const.
const
```

## 二、常用数据类型(结构)

### 2.1 整数和浮点数

作为直接量直接使用，整数相除即使能够除尽得到的结果仍然是浮点数，想获取除数可以使用`div`操作符，想获取余数可以使用`rem`操作符。

```
5> 5 / 3.
1.6666666666666667
6> 5 div 3.
1
7> 5 rem 3.
2
8> 4 / 2.
2.0
```

Erlang在内部使用64位的IEEE 754-1985浮点数，因此浮点数的程序会存在和C等语言一样的浮点数取整和精度问题。

### 2.2 原子

原子类似于C语言中的宏，被用于表示常量值，以小写字母开头，后面可以跟字母、数字、下划线或at符号

```
9> helloworld.
helloworld
```

### 2.3 元组

元组是由一些**数量固定**的项目归组形成的单一的实体，创建元组的方法就是用大括号将想要表示的值括起来，并用逗号分隔每一部分。类似于C语言中的结构体，C语言中的结构体如下定义

```
struct point {
    int x;
    int y;
} P;
```

而对应的元组如下定义

```
10> P = {1, 2}.
{1,2}
11> P.
{1,2}
```

为了指明元组的含义，一般情况下将第一个元素定义为原子类型，如下

```
12> Q = {point, 1, 2}.
{point,1,2}
13> Q.
{point,1,2}
```

元组还可以嵌套，例如将如下JSON串利用嵌套的元组表示

```
{
  "name" : "joe",
  "height" : 1.82,
  "footsize" : 42,
  "eyecolour" : "brown"
}
```

```
14> Person = {person, {name, joe}, {height, 1.82}, {footsize, 42}, {eyecolour, b
rown}}.
{person,{name,joe},
        {height,1.82},
        {footsize,42},
        {eyecolour,brown}}
15> Person.
{person,{name,joe},
        {height,1.82},
        {footsize,42},
        {eyecolour,brown}}
```

其实可以看出，把元组大括号的第一个元素定义为常量，就相当于键值对中键的作用，但是并不是可以通过键提取值，而是代表了值的含义是什么。

但是如何提取元组的值呢？答案是通过模式匹配操作符`=`来实现数据绑定，如下

```
16> Point = {point, 10, 45}.
{point,10,45}
20> {point, First, Second} = Point.
{point,10,45}
21> First.
10
22> Second.
45
```

如果不匹配，比如说参数个数不一致，常量绑定值或者其他情况，那么就会提示错误，如下

```
17> {point, X, Y} = Point.
** exception error: no match of right hand side value {point,10,45}
18> {Point, First, second} = Point.
** exception error: no match of right hand side value {point,10,45}
19> {Point, First, Second} = Point.
** exception error: no match of right hand side value {point,10,45}
20> {point, First, Second} = Point.
```

再试试提取复杂元组的值，需要注意的是，如果我们对元组当中某个值不感兴趣，那么可以通过占位符`_`将其忽略，这样也省得定义没必要的变量名，如下提取前面定义的Person的元组值

```
24> {_, {_, Name}, {_, Height}, {_, Footsize}, {_, Eyecolour}} = Person.
{person,{name,joe},
        {height,1.82},
        {footsize,42},
        {eyecolour,brown}}
25> Name.
joe
26> Height.
1.82
28> Footsize.
42
29> Eyecolour.
brown
```

怎么样修改值呢？不会的，因为Erlang的变量是不允许修改的！

### 2.3 列表

列表是用来存放任意数量的事物的，创建列表的方法就是用中括号将列表元素括起来，并用逗号分隔它们，如下

```
30> Drawing = [{square, {10, 10}, 10}, {triangle, {15, 10}, {25, 10}, {30, 40}}]
.
[{square,{10,10},10},{triangle,{15,10},{25,10},{30,40}}]
31> Drawing.
[{square,{10,10},10},{triangle,{15,10},{25,10},{30,40}}]
```

列表可以分为表头和表尾，表头是第一个元素，除过表头的元素全部都是表尾，在多元素的情况下，表尾很有可能是一个列表。由于rlang不支持修改变量，那么拓展列表的办法就是将其赋值给新的变量并定义拓展的元素，如下

```
32> ThingToBuy = [{apples, 10}, {pears, 6}, {milk, 3}].
[{apples,10},{pears,6},{milk,3}]
33> ThingToBuy1 = [{oranges, 4}, {newspaper, 1} | ThingToBuy].
[{oranges,4},{newspaper,1},{apples,10},{pears,6},{milk,3}]
```

列表元素的提取仍然是通过模式匹配操作符`=`来实现数据绑定，不过对于列表，我们可以通过表头表尾的配合来提取元素，如下

```
34> [Buy1 | ThingToBuy2] = ThingToBuy1.
[{oranges,4},{newspaper,1},{apples,10},{pears,6},{milk,3}]
35> Buy1.
{oranges,4}
36> ThingToBuy2.
[{newspaper,1},{apples,10},{pears,6},{milk,3}]
```

上述代码是将表头与表尾分别提取，可以以一次性提取若干的表头，以提取两个表头为例

```
37> [Buy2, Buy3 | ThingToBuy3] = ThingToBuy1.
[{oranges,4},{newspaper,1},{apples,10},{pears,6},{milk,3}]
38> Buy2.
{oranges,4}
39> Buy3.
{newspaper,1}
40> ThingToBuy3.
[{apples,10},{pears,6},{milk,3}]
```

占位符`_`忽略元素同样有用，我们以下忽略列表的前四个元素，只提取最后一个元素

```
42> [_, _, _, _ | Buy4] = ThingToBuy1.
[{oranges,4},{newspaper,1},{apples,10},{pears,6},{milk,3}]
43> Buy4.
[{milk,3}]
```

### 2.4 字符串

严格来说，Erlang中并没有字符串，想要在Erlang中表示字符串，可以选择由一个整数组成的列表或者一个二进制型。当字符串表示为一个整数列表时，列表中的每个元素都代表了一个Unicode代码点。

创建字符串最简单的方式就是使用字符串字面量，如下

```
44> Names = "Hello, World".
"Hello, World"
```

其实上述并不是真的字符串，而只是一个列表的简写，这个列表中包含了每个字符的整数字符代码，不信试试看

可以通过操作符`$`获取某个字符的整数字符代码，如下

```
45> H = $H.
72
46> E = $e.
101
47> L = $l.
108
48> O = $o.
111
49> Sign = $,.
44
50> Space = $ .
32
51> W = $W.
87
52> R = $r.
114
53> D = $d.
100
54> HelloWorld = [H, E, L, L, O, Sign, Space, W, O, R, L, D].
"Hello, World"
```
可见，效果一模一样。