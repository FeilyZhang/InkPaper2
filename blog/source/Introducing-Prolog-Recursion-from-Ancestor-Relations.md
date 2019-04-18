title: "由祖先关系引入Prolog递归"
date: 2019-02-04 11:03:06 +0800
update: 2019-02-04 11:03:06 +0800
author: me
cover: "-/images/swipl.png"
tags:
    - Prolog
preview: 声明式编程语言通常是用一种称为“尾递归优化”的技术来解决递归调用引起栈溢出的问题。也就是说如果你将一个递归的子目标放到递归规则的末尾，Prolog会通过丢弃调用栈来优化这次调用，并保持内存占用不变。所以我们的ancestor(Z, Y)递归子目标是放在递归规则的末尾的。

---

我们先看一组父子关系

```
father(zeb, john_boy_sr).
father(john_boy_sr, john_boy_jr).
```

以上代码描述，zeb是john_boy_sr的父亲，而john_boy_sr又是john_boy_jr的父亲，我们可以直观的发现上述关系存在三组祖先关系，如下

+ zeb是john_boy_sr的祖先
+ john_boy_sr是john_boy_jr的祖先
+ zeb是john_boy_jr的祖先

如果我们用Prolog来推理这三组祖先关系会怎么样呢？我们先写第一个推理规则，使得以上事实在该规则约束下得到祖先关系的结论，如下

```
ancestor(X, Y) :-
    father(X, Y).
```

推理结果为

```
?- ancestor(Who1, Who2).
Who1 = zeb,
Who2 = john_boy_sr ;
Who1 = john_boy_sr,
Who2 = john_boy_jr.
```

可见只推理出了两组直观的祖先关系，就是我们的原有事实，但是并未推理出两组事实的传递关系。纳闷我们再改一下规则约束，让祖先关系可以传递，如下

```
ancestor(X, Y) :-
    father(X, Z), father(Z, Y).
```

以上规则表明如果X是Z的祖先，Z是Y的祖先，那么X就是Y的祖先(就是推理结论)，推理结果如下

```
?- ancestor(Who1, Who2).
Who1 = zeb,
Who2 = john_boy_jr ;
false.
```

这次传递关系出来了，但是第一次的最直观的两组祖先关系又不见了。

现在可以得出，我们最终的结果应该是第一条规则和第二条规则的并集，那么我们只需要这样写规则即可

```
ancestor(X, Y) :-
    father(X, Y).

ancestor(X, Y) :-
    father(X, Z), father(Z, Y).
```

推理结果为

```
?- ancestor(Who1, Who2).
Who1 = zeb,
Who2 = john_boy_sr ;
Who1 = john_boy_sr,
Who2 = john_boy_jr ;
Who1 = zeb,
Who2 = john_boy_jr ;
false.
```

这次推理结果全部出来了。可喜可贺。但是如果我们再增加一层传递关系呢(直接的关系肯定可以得出结论，因为本身就满足father关系)？我们的规则还能原封不动的推理出所有的结论吗？不妨试试,代码如下

```
father(zeb, john_boy_sr).
father(john_boy_sr, john_boy_jr).
father(john_boy_jr, john_boy_br).

ancestor(X, Y) :-
    father(X, Y).

ancestor(X, Y) :-
    father(X, Z), father(Z, Y).
```

推理结果为

```
?- ancestor(Who1, Who2).
Who1 = zeb,
Who2 = john_boy_sr ;
Who1 = john_boy_sr,
Who2 = john_boy_jr ;
Who1 = john_boy_jr,
Who2 = john_boy_br ;
Who1 = zeb,
Who2 = john_boy_jr ;
Who1 = john_boy_sr,
Who2 = john_boy_br ;
false.
```

可以看出，这种传递关系我们的推理不完备的，还缺一个结论，即

```
Who1 = zeb,
Who2 = john_boy_br ;
```

这是因为我们的规则并没有体现出这样的关系，想要体现这样的关系，那么我们还需要并集一下另一个规则，如下


```
father(zeb, john_boy_sr).
father(john_boy_sr, john_boy_jr).
father(john_boy_jr, john_boy_br).

ancestor(X, Y) :-
    father(X, Y).

ancestor(X, Y) :-
    father(X, Z), father(Z, Y).
	
ancestor(X, Y) :-
    father(X, Z), father(Z, B), father(B, Y).
```

这样，我们的推论就是完全的，为

```
?- ancestor(Who1, Who2).
Who1 = zeb,
Who2 = john_boy_sr ;
Who1 = john_boy_sr,
Who2 = john_boy_jr ;
Who1 = john_boy_jr,
Who2 = john_boy_br ;
Who1 = zeb,
Who2 = john_boy_jr ;
Who1 = john_boy_sr,
Who2 = john_boy_br ;
Who1 = zeb,
Who2 = john_boy_br ;
false.
```

上述示例是一种直接的传递关系,即首尾相连，我们得出的结论是关系的增加也就意味着我们的规则也要做出相应的改变，不然推论就是不完全的。

其实还有另一种传递关系，即部分首尾不相连，关系如下

```
father(zeb, john_boy_sr).
father(john_boy_sr, john_boy_jr).
father(zeb, john_boy_br).
father(john_boy_br, john_boy_sr).
```

相应的代码如下

```
father(zeb, john_boy_sr).
father(john_boy_sr, john_boy_jr).
father(zeb, john_boy_br).
father(john_boy_br, john_boy_sr).

ancestor(X, Y) :-
    father(X, Y).

ancestor(X, Y) :-
    father(X, Z), father(Z, Y).
```

推理结果为

```
?- ancestor(Who1, Who2).
Who1 = zeb,
Who2 = john_boy_sr ;
Who1 = john_boy_sr,
Who2 = john_boy_jr ;
Who1 = zeb,
Who2 = john_boy_br ;
Who1 = john_boy_br,
Who2 = john_boy_sr ;
Who1 = zeb,
Who2 = john_boy_jr ;
Who1 = zeb,
Who2 = john_boy_sr ;
Who1 = john_boy_br,
Who2 = john_boy_jr.
```

我们的推理结果没毛病。因为我们我们的规则已经囊括了所有的可能性。如果我们越增加关系的复杂程度，那么我们现在的程序就不是健壮的，就无法推理出所有的结论。

我们必须找到一个合适的办法才行，保证我们不管怎样增加关系，我们的规则约束同样适用。实际上，递归是很好的解决办法。我们再回顾一下我们的代码，如下

```
father(zeb, john_boy_sr).
father(john_boy_sr, john_boy_jr).
father(zeb, john_boy_br).
father(john_boy_br, john_boy_sr).

ancestor(X, Y) :-
    father(X, Y).

ancestor(X, Y) :-
    father(X, Z), father(Z, Y).
```

在第二个ancestor中，我们用且关系链接了两个father，如果我们把第二个father用ancestor()替换，那么就是一个递归的例子，因为father(X, Y)的推论是ancestor(X, Y)。而我们最终的推论也是推ancestor（X, Y）,所以两个ancestor的方向是一致的，代码如下

```
father(zeb, john_boy_sr).
father(john_boy_sr, john_boy_jr).
father(zeb, john_boy_br).
father(john_boy_br, john_boy_sr).

ancestor(X, Y) :-
    father(X, Y).

ancestor(X, Y) :-
    father(X, Z), ancestor(Z, Y).
```

推理结果为

```
?- ancestor(Who1, Who2).
Who1 = zeb,
Who2 = john_boy_sr ;
Who1 = john_boy_sr,
Who2 = john_boy_jr ;
Who1 = zeb,
Who2 = john_boy_br ;
Who1 = john_boy_br,
Who2 = john_boy_sr ;
Who1 = zeb,
Who2 = john_boy_jr ;
Who1 = zeb,
Who2 = john_boy_sr ;
Who1 = zeb,
Who2 = john_boy_jr ;
Who1 = john_boy_br,
Who2 = john_boy_jr ;
false.
```

其实有一组结果推理了两次，但是结果是完备的。

那么能不能将第二个ancestor的第一个father()改成ancestor或者将两个father全部改了呢？不可以。改了之后推理过程中会栈溢出

```
?- ancestor(Who1, Who2).
Who1 = zeb,
Who2 = john_boy_sr ;
Who1 = john_boy_sr,
Who2 = john_boy_jr ;
Who1 = zeb,
Who2 = john_boy_br ;
Who1 = john_boy_br,
Who2 = john_boy_sr ;
Who1 = zeb,
Who2 = john_boy_jr ;
;ERROR: Stack limit (1.0Gb) exceeded
ERROR:   Stack sizes: local: 1.0Gb, global: 15Kb, trail: 2Kb
ERROR:   Stack depth: 12,200,666, last-call: 0%, Choice points: 6
ERROR:   Probable infinite recursion (cycle):
ERROR:     [12,200,666] user:ancestor(john_boy_jr, _3912)
ERROR:     [12,200,665] user:ancestor(john_boy_jr, _3932)
```

因为每个递归的子目标都会使用栈空间，最终就有可能耗尽栈空间。但是为什么递归语句放到最后不存在问题呢？

声明式编程语言通常是用一种称为“尾递归优化”的技术来解决递归调用引起栈溢出的问题。也就是说如果你将一个递归的子目标放到递归规则的末尾，Prolog会通过丢弃调用栈来优化这次调用，并保持内存占用不变。所以我们的ancestor(Z, Y)递归子目标是放在递归规则的末尾的。