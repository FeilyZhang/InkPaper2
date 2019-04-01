title: "剖析StringBuilder类"
date: 2018-12-18 16:45:06 +0800
update: 2018-12-18 16:45:06 +0800
author: me
cover: "-/images/java.jpg"
tags:
    - Java Core
preview: String类适合少量字符串的不太频繁的操作，因为String的每次的字符串操作基本上都是重新创建一个字符数组，这样性能太低，可以考虑使用StringBuilder或者StringBuffer。

---

String类适合少量字符串的不太频繁的操作，因为String的每次的字符串操作基本上都是重新创建一个字符数组，这样性能太低，很多时候我们需要兼顾性能，能不能直接在字符串上 操作而不是拷贝到另一个字符数组再操作？

答案就是用StringBuilder或者StringBuffer，这两个类的方法基本上都是一样的实现的代码也几乎一样，唯一不同的地方在于StringBuffer是线程安全的，而StringBuilder是线程不安全的。线程安全的成本就是性能耗损，所以在不存在线程安全的情况下StringBuilder处理大量字符串无疑是首选

## 基本用法

创建StringBuilder对象

```
StringBuilder ab = new StringBuilder();
sb.append(String str);
```

然后通过toString获取字符串

```
System.out.println(sb.toString());
```

## 基本实现原理

StringBuilder内部仍然是基于字符数组，但是该字符数组不是final的，是可以修改的，与String不同的是，字符数组中不一定所有位置都已经被使用，它有一个实例变量，表示数组中已经使用的字符个数,该实例变量如下定义

```
int count;
```

StringBuilder继承自AbstractStringBuilder，它的默认构造方法是

```
public StringBuilder() {
    super(16);
}
```

即调用父类的构造方法，其父类的构造方法为

```
AbstractStringBuilder(int capacity) {
    value = new cahr[capacity];
}
```

即创建一个容量为16的字符数组。

append方法的实现为

```
public AbstractStringBuilder append(String str) {
    if (str == null) str = "null";
    int len = str.length;
    ensureCapacityInternal(count + len);
    str.getChars(0, len, value, count);
    count += len;
    return this;
}
```

可见，append会直接复制字符到内部的字符数组中，如果字符数组的长度不够，那么会进行拓展，并增加实际使用的长度。

toString方法的实现为

```
public String toString() {
    return new String(value, o, count);
}
```

即直接将字符数组转化为字符串。

## String的+和+=运算符

String的+和+=运算符，是Java编译器提供的支持，但是背后会转化为StringBuilder的append方法，所以在大量且频繁的字符串操作时应该尽量避免使用String的+和+=运算符，性能比较低。