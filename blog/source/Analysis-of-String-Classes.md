title: "剖析String类"
date: 2018-12-18 16:16:08 +0800
update: 2018-12-18 16:16:08 +0800
author: me
cover: "-/images/java.jpg"
tags:
    - Java Core
preview: String类基于char数组，String对字符串的操作并不是在字符串或者字符数组本身进行操作，而是创建一个新的char数组然后把内容复制进去再操作。

---

String类的常用方法：

```
public boolean isEmpty()
public int length
public String subString(int beginIndex)
public String subString(int beginIndex, int endIndex)
public int indexOf(char c)
public int indexOf(String str)
public int lastIndexOf(char c)
public int lattIndexOf(String str)
public boolean contains(CharSequence s)
public boolean startsWith(String str)
public boolean endsWith(String str)
public boolean equals(Object obj)
public boolean equalsIgnoreCase(String str)
public int compareTo(String anotherString)
public int compareToIgnoreCase(String str)
public String toUpperCase()
public String toLowerCase()
public String concat(String str)
public String replace(char oldChar, char newChar)
public String replace(CharSequence target, CharSequence replacement)
public String trim()
```

## String类的内部

String类基于char数组，有两个构造方法可以根据char数组创建String变量

```
public String(char[] value)
public String(char[] value, int offset, int count)
```

String对字符串的操作并不是在字符串或者字符数组本身进行操作，而是创建一个新的char数组然后把内容复制进去再操作

Java使用Charset类表示各种编码，它有两个常用静态方法

```
public static Charset defaultCharset() //获取系统默认编码
public static Charset forName(String charsetName) //设置编码
```

## String类的不可变性

与包装类相似，String类也是不可变类，即对象一旦创建，就无法修改。String类也被声明为final，无法继承，内部的char数组也是final的，一旦初始化就不能再变。这也就是说String内部对字符串进行操作的方法是通过拷贝原有字符串再操作实现的。

## String常量字符串

String常量字符串被存放在JVM常量池当中，也就是说就算创建多份相同的常量，实际上这些常量都是引用的同一份JVM常量池的常量，测试一下

```
public class StringTest {

    public static void main(String[] args) {
        String str1 = "hello,world";
        String str2 = "hello,world";
        System.out.println(str1 == str2); //true
    }

}
```

以上定义的是两个常量，由于常量存放在常量池，所以二者的引用是一致的，所以打印true，上面的代码类似于

```
public class StringTest {

    public static void main(String[] args) {
        String str = new String(new char[] {'h','e','l','l','o',',','w','o','r','l','d'});
        String str1 = str;
        String str2 = str;
        System.out.println(str1 == str2);
    }

}
```

即实际上只有一个对象，那么当然等于自身

如果用new创建，那么就不会存放在常量池，而是Java堆，如下

```
public class StringTest {

    public static void main(String[] args) {
        String str1 = new String("hello,world");
        String str2 = new String("hello,world");
        System.out.println(str1 == str2); //false
    }

}
```

所以输出为false，因为有两个对象。上面的代码类似于

```
public class StringTest {

    public static void main(String[] args) {
        String str = new String(new char[] {'h','e','l','l','o',',','w','o','r','l','d'});
        String str1 = new String(str);
        String str2 = new String(str);
        System.out.println(str1 == str2);
    }

}
```

而String类中以String为参数的构造方法的代码如下

```
public String(String str) {
    this.value = str.value;
    this.hash = str.hash;
}
```

str1与str2的value值共同指向常量池的同一个常量，但是hash不同，所以二者不同。

如果用equals方法比较，那么输出为真

```
public class StringTest {

    public static void main(String[] args) {
        String str1 = new String("hello,world");
        String str2 = new String("hello,world");
        System.out.println(str1.equals(str2)); //true
    }

}
```

这是由于equals方法比较对象内容是否相等，而==只是比较常量的引用，后者速度更快。