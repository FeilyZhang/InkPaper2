title: "Arrays类的相关用法"
date: 2018-12-18 17:09:05 +0800
update: 2018-12-18 17:09:05 +0800
author: me
cover: "-/images/java.jpg"
tags:
    - Java Core
preview: toString方法, sort方法, binarySearch()方法。

---

数组的Arrays类有很多有用的功能，看一下

## toString方法

如果我们想直接输出整个数组，如果我们像下面这样的话得到的只是数组的内存地址，而Arrays.toString()方法得到的就是预期结果

```
import java.util.Arrays;

public class ArraysTest {

    public static void main(String[] args) {
        int[] array = {2， 3， 1};
        System.out.println(array);  //[I@15db9742
        System.out.println(Arrays.toString(array)); //[1, 2, 3]
    }

}
```

## sort方法

对于除boolean类型外的数组，Arrays均提供了排序方法

int型数组从小到大排序为

```
import java.util.Arrays;

public class ArraysTest {

    public static void main(String[] args) {
        int[] array = {1, 2, 3};
        Arrays.sort(array);
        System.out.println(Arrays.toString(array)); //[1, 2, 3]
    }

}
```

String型数组的排序为

```
import java.util.Arrays;

public class ArraysTest {

    public static void main(String[] args) {
        String[] str = {"Feily", "Zhang", "Haoyue", "Li", "hello", "world"};
        Arrays.sort(str);
        System.out.println(Arrays.toString(str)); //[Feily, Haoyue, Li, Zhang, hello, world]
    }

}
```

是按照首字母的ASCII码int值排序的，由于大些字母的ASCII码小于小写，所以在前面，如果排序像忽略大小写，那么如下

```
import java.util.Arrays;

public class ArraysTest {

    public static void main(String[] args) {
        String[] str = {"Feily", "Zhang", "Haoyue", "Li", "hello", "world"};
        Arrays.sort(str, String.CASE_INSENSITIVE_ORDER);
        System.out.println(Arrays.toString(str)); //[Feily, Haoyue, hello, Li, world, Zhang]
    }

}
```

## binarySearch()方法

```
import java.util.Arrays;

public class ArraysTest {

    public static void main(String[] args) {
        int[] arr = {5, 3, 1, 4, 2};
        String[] str = {"Feily", "Zhang", "Haoyue", "Li", "hello", "world"};
        System.out.println(Arrays.binarySearch(str, "Li")); //3
        Arrays.sort(arr);
        System.out.println(Arrays.binarySearch(str, "hello")); //4
        System.out.println(Arrays.binarySearch(arr, 5));    //-6
        Arrays.sort(arr);
        System.out.println(Arrays.binarySearch(arr, 5));    //4
    }

}
```
在二分查找整形数组时一定要保证元素有序，否则不会达到预期效果，字符串数组的话应该都可以。如果找不到指定的元素，那么就会返回负数，负数等于插入点+1，插入点指的是在这个位置插入没找到的元素仍然可以保证数组有序。