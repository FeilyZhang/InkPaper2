title: "地图着色问题"
date: 2019-02-14 13:56:06 +0800
update: 2019-02-14 13:56:06 +0800
author: me
cover: "-/images/article/map-color.png"
tags:
    - Prolog
preview: 现在有一幅地图，需要进行着色，为了区分A, B, C, D, E五个区域，要求相邻区域颜色不一样，并且颜色只有红绿蓝三种，那么要怎样定义事实描述和规则约束呢？

---

现在有一幅地图，如下

![](/images/article/map-color.png)

需要进行着色，为了区分A, B, C, D, E五个区域，要求相邻区域颜色不一样，并且颜色只有红绿蓝三种，那么要怎样定义事实描述和规则约束呢？

我们先定义事实描述，就是用到的三种颜色,同样是使用常量进行具体的定义
```
color(red).
color(green).
color(blue).
```
满足结论的约束规则是红绿蓝三种颜色且相邻区域颜色不一样。同样是使用大写变量进行笼统的定义

我们先定义结论和红绿蓝的关系，如下
```
colorify(A, B, C, D, E) :-
    color(A), color(B), color(C), color(D), color(E).
```
以上代码的含义是A、B、C、D、E分别代表各区域的颜色(即代表了区域也代表了颜色)，取值是red、blue、green的其中一种，分别代表5个区域的地图颜色。

以上代码只是解决了地图有颜色的问题，但是并没有解决颜色对不对的问题。我们接下来要解决这个问题。

通过观察地图我们发现

	- 区域A与区域B、C、D、E相接壤，那么也就是说A的颜色与B、C、D、E的颜色不一样，即\+ A = B, \+ A = C, \+ A = D, \+ A = E,
	- 区域B和C相接壤，也就是B和C的颜色不一样，即\+B = C,
	- 区域C和D相接壤，也就是C和D的颜色不一样，即\+C = D,
	- 区域D和E相接壤，也就是D和E的颜色不一样，即\+E = E.

那么现在综合以上规则，如下
```
colorify(A,B,C,D,E) :-
    color(A), color(B), color(C), color(D), color(E),
    \+ A = B, \+ A = C, \+ A = D, \+ A = E,
    \+ B = C, \+ C = D, \+ D = E.
```
再综合事实描述，代码为
```
color(red).
color(green).
color(blue).

colorify(A,B,C,D,E) :-
    color(A), color(B), color(C), color(D), color(E),
    \+ A = B, \+ A = C, \+ A = D, \+ A = E,
    \+ B = C, \+ C = D, \+ D = E.
```
shell运行，不断追问，有如下输出
```
Welcome to SWI-Prolog (threaded, 64 bits, version 8.0.1)
SWI-Prolog comes with ABSOLUTELY NO WARRANTY. This is free software.
Please run ?- license. for legal details.

For online help and background, visit http://www.swi-prolog.org
For built-in help, use ?- help(Topic). or ?- apropos(Word).

?- colorify(A, B, C, D, E).
A = red,
B = D, D = green,
C = E, E = blue ;
A = red,
B = D, D = blue,
C = E, E = green ;
A = green,
B = D, D = red,
C = E, E = blue ;
A = green,
B = D, D = blue,
C = E, E = red ;
A = blue,
B = D, D = red,
C = E, E = green ;
A = blue,
B = D, D = green,
C = E, E = red ;
false.

?- 
```
可见Prolog共给出了6种结论。

其实，不难发现，结论也是一种事实，结论这种事实是原有事实在规则约束下得到的。这个过程就是推论