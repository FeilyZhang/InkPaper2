title: "Java中的数据类型转换"
date: 2018-11-15 00:23:03 +0800
update: 2018-11-15 00:23:03 +0800
author: me
cover: "-/images/java.jpg"
tags:
    - Java Core
preview: Java中的数据类型分为三大类，分别是数值型、布尔型、字符型，其中数值型分为整型和浮点型。

---

Java中的数据类型分为三大类，分别是数值型、布尔型、字符型，其中数值型分为整型和浮点型。

相对于数据类型，Java中的变量类型分为布尔型boolean；字符型char；整型byte、short、int、long；浮点型float、double。

需要注意的是String虽然平时与int、double等常用，但是String并不是Java当中的简单数据类型，String是类变量。

Java数据类型的转换分为三种，分别是：简单数据类型之间的转换(低级变量向高级变量的自动类型转换、高级变量向低级变量的强制类型转换以及包装类过渡类型转换)、字符串型与其他数据类型的转换、将字符型直接作为数值转换为其他数据类型。

Java当中简单数据类型由低级到高级分别是：(byte、short、char)—int—long—float—double

## 一、简单数据类型之间的转换

### 1.1 低级变量向高级变量的自动类型转换

```
public class TestPlus {
    public static void main(String[] args) {
        byte bi = 10;
        int ii = bi;
        long li = ii;
        float fi = li;
        double di = fi;
        System.out.println(bi);    //10
        System.out.println(ii);    //10
        System.out.println(li);    //10
        System.out.println(fi);    //10.0
        System.out.println(di);    //10.0
        char cj = 'a';
        int ij = cj;
        long lj = ij;
        float fj = lj;
        double dj = fj;
        System.out.println(cj);    //a
        System.out.println(ij);    //97
        System.out.println(lj);    //97
        System.out.println(fj);    //97.0
        System.out.println(dj);    //97.0

    }
}
```

如果是char型自动类型转化那么会转化为其对应的ASCII码值。

对于char、byte、short三种类型而言，由于他们是相同的级别，所以不能自动类型转化，但是可以强制类型转化，比如

```
byte bi = 97;
System.out.println((char)bi);    //a
```

### 1.2 高级变量向低级变量的强制类型转换

```
public class TestPlus {
    public static void main(String[] args) {
        double di = 10.0;
        float fi = (float)di;
        long li = (long)di;
        int ii = (int)di;
        byte bi = (byte)ii;
        System.out.println(bi);    //10
        System.out.println(ii);    //10
        System.out.println(li);    //10
        System.out.println(fi);    //10.0
        System.out.println(di);    //10.0
    }
}
```

不过这种转化会导致溢出或精度下降，比如

```
public class TestPlus {
    public static void main(String[] args) {
        double di = 10.1111111111111111111;
        float fi = (float)di;
        long li = (long)di;
        int ii = (int)di;
        byte bi = (byte)ii;
        System.out.println(bi);    //10
        System.out.println(ii);    //10
        System.out.println(li);    //10
        System.out.println(fi);    //10.111111
        System.out.println(di);    //10.11111111111111
    }
}
```

明显损失掉了double型变量的精度

### 1.3 包装类过渡类型转换

包装类其实就是对简单数据类型的一个包装,char、int、long、float、double、boolean类型的包装类分别是Character、Integer、Long、Float、Double、Boolean，而String本身就是类因此不存在包装类。

对简单数据类型进行包装的一个好处就是规避了NPE风险，简单数据类型是不可以接收null值的，比如

```
double di = null;    //Error:(5, 21) java: incompatible types: <nulltype> cannot be converted to double
```

以上代码会报错，但是包装类却不存在问题

```
Double di = null;
System.out.println(di);    //null
```

在进行简单数据类型之间的转换(无论自动还是强制)时可以用包装类进行中间过渡，如下

```
Double di = 10.11111111111;
System.out.println(di);    //10.11111111111
System.out.println(di.floatValue());    //10.111111
```

直接得到了double对应的包装类Double转化的float型数据

简单数据类型到包装数据类型(装箱的过程)可以通过相应包装类的构造函数，例如

```
Double di = new Double(10.11111111111);
Float fi = new Float(10.1111f);
Long li = new Long(10l);
Integer ii = new Integer(10);
Boolean bi = new Boolean(true);
Character ci = new Character('a');
```

而拆箱同样简单，只需要将包装类对象直接赋值给相应的简单类型变量即可。

## 二、字符串型与其他数据类型的转换

### 2.1 其它数据类型转字符串，通过相应包装类的toString()方法即可实现

```
public class TestPlus {
    public static void main(String[] args) {
        Double di = new Double(10.11111111111);
        Float fi = new Float(10.1111f);
        Long li = new Long(10l);
        Integer ii = new Integer(10);
        Boolean bi = new Boolean(true);
        Character ci = new Character('a');
        System.out.println(di.toString().getClass().toString());
        System.out.println(fi.toString().getClass().toString());
        System.out.println(li.toString().getClass().toString());
        System.out.println(ii.toString().getClass().toString());
        System.out.println(bi.toString().getClass().toString());
        System.out.println(ci.toString().getClass().toString());
    }
}
```

如下输出

```
class java.lang.String
class java.lang.String
class java.lang.String
class java.lang.String
class java.lang.String
class java.lang.String
```

可以看到明显转化为了String类型，需要注意的是只有包装类的toString方法可以getClass，因为toString的目标类型是String类，那么就可以通过getClass获取其类型；但是诸如包装类的floatValue之类的转化附带拆箱的方法是无法getClass的，因为其目标类型是简单数据类型，简单数据类型不是类，无法getClass

### 2.2 字符串转简单数据类型，通过包装类的parse方法即可实现

```
String s = "11";
System.out.println(Integer.parseInt(s));    //11
System.out.println(Double.parseDouble(s));    //11.0
```

parse之后的目标类型是简单数据类型(附带拆箱)，所以仍然无法getClass

### 2.3 将字符型直接作为数值转换为其他数据类型

**一种是将字符型直接转化为其对应的ASCII码值，**如

```
char c = '1';
System.out.println((int)c);    //49
```

<a name="d9c2dd1a"></a>
**另一种是直接获取字面量**

但是我们有些时候想得到的是字面量1的数值(是数值)，那么可以通过Character的getNumericValue方法实现，如下

```
char c = '1';
System.out.println(Character.getNumericValue(c));    //1
```

全文完！

