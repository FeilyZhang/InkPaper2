title: "泛型方法与泛型接口"
date: 2018-12-08 17:28:45 +0800
update: 2018-12-08 17:28:45 +0800
author: me
cover: "-/images/java.jpg"
tags:
    - Java Core
preview: 泛型不仅仅是类层面的，也可以单独给方法定义，形式是在方法的声明的返回值类型前加上`<T>`，这是只有一种泛型的情况，如果是多种泛型，那么用逗号隔开即可，然后就可以在形参列表使用了。

---

泛型不仅仅是类层面的，也可以单独给方法定义，形式是在方法的声明的返回值类型前加上`<T>`，这是只有一种泛型的情况，如果是多种泛型，那么用逗号隔开即可，然后就可以在形参列表使用了。如下，我们实现一个数组的相关方法

## 一、泛型方法

```
public class Array {

    public static  int indexFirstOf(T[] array, T ele) {
        for (int i = 0; i < array.length; i++) { 
            if (array[i].equals(ele)) { 
                return i; 
            } 
        } 
        return -1; 
    } 

    public static  int indexLastOf(T[] array, T ele) {
        for (int i = array.length - 1; i >= 0; i--) {
            if (array[i].equals(ele)) {
                return i;
            }
        }
        return -1;
    }
    
    public static  boolean isExist(T[] array, T ele) {
        for (T e : array) {
            if (e.equals(ele)) {
                return true;
            }
        }
        return false;
    }
    
}
public class ArrayTest {

    public static void main(String[] args) {
        String[] string = {"Zhang", "Peng", "Fei", "Zhang"};
        System.out.println(Array.indexFirstOf(string, "Zhang"));
        System.out.println(Array.indexLastOf(string, "Zhang"));
        System.out.println(Array.isExist(string, "Zhang"));
        System.out.println(Array.isExist(string, "feily"));
    }

}
```

输出为

```
0
3
true
false
```

## 二、泛型接口

接口也可以是泛Comparator型的，注意是接口层面而不是接口中的抽象方法，Comparable和Comparator接口都是泛型的，代码如下

```
public interface Comparable {
    public int comparaTo(T o);
}
public interface comparator {
    int compare(T o1, T o2);
    boolean equals(Object obj);
}
```

实现接口时，应该指定具体的类型，比如，对于Integer类，实现代码是

```
public final class Integer extends Number implements Comparable {
    public int comparaTo(Integer anotherInteger) {
	      return compare(this.value, anotherInteger.value);
    }
}
```

而String类内部的一个Comparator接口实现为

```
private static class CaseInsensitiveComparator implements Comparator {
    public int compare(String s1, String s2) {
        // ...
    }
}
```