title: "Java函数式编程之Predicate接口的用法"
date: 2018-12-09 11:41:31 +0800
update: 2018-12-09 11:41:31 +0800
author: me
cover: "-/images/java.jpg"
tags:
    - Java Core
preview: Java8定义了大量的预定义函数式接口，用于常见的代码传递(其实就是参数化函数)，这些函数定义在包java.util.function下,每个接口只有一个抽象方法.

---

Java8定义了大量的预定义函数式接口，用于常见的代码传递(其实就是参数化函数)，这些函数定义在包java.util.function下,每个接口只有一个抽象方法

Predicate函数接口的接口签名如下

```
Predicate<T>
```

其唯一的方法test的签名如下

```
boolean test(T t)    //谓词，判断输入是否满足条件
```

用一个例子来简单说明，首先定义序列化学生姓名和成绩的JavaBean，如下

```
public class Student {

    String name;
    double score;
    
    public Student(String name, double score) {
        this.name = name;
        this.score = score;
    }
    
    public void setName(String name) {
        this.name = name;
    }
    
    public void setScore(double score) {
        this.score = score;
    }
    
    public String getName() {
        return name;
    }
    
    public double getScore() {
        return score;
    }
}
```

再在main方法所在的主文件中重写有Predicate和test方法参与的函数，如下

```
public static <E> List<E> filter(List<E> list, Predicate<E> pred) {
    List<E> retList  = new ArrayList<E>();
    for (E e : list) {
        if (pred.test(e)) {
            retList.add(e);
        }
    }
    return retList;
}
```

该方法通过Predicate的test过滤不符合条件的Student，正是谓词的作用

那么再定义Student型List列表，通过arrays.asList方法将Student型数组转化为列表，如下

```
List<Student> list = Arrays.asList(new Student[] {
    new Student("Feily Zhang", 95d), new Student("Haoyue Li", 100d),
    new Student("Daji Luobu", 80d), new Student("Xin Chen", 94d)
});
```

然后就需要调用filter方法，传入list，并编写Lambda表达式作为函数接口Predicate的实现，如下

```
List<Student> result = filter(list, student -> student.getScore() > 90);
```

这样过滤的结果就会全部保存在result列表中

完整的主文件代码如下

```
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;
import java.util.function.Predicate;

public class StudentTest {

    public static void main(String[] args) {
        List<Student> list = Arrays.asList(new Student[] {
            new Student("Feily Zhang", 95d), new Student("Haoyue Li", 100d),
            new Student("Daji Luobu", 80d), new Student("Xin Chen", 94d)
        });
        List<Student> result = filter(list, student -> student.getScore() > 90);
        for (Student ele : result) {
            System.out.println(ele.getName() + " , " + ele.getScore());
        }
    }

    public static <E> List<E> filter(List<E> list, Predicate<E> pred) {
        List<E> retList  = new ArrayList<E>();
        for (E e : list) {
            if (pred.test(e)) {
                retList.add(e);
            }
        }
        return retList;
    }
}
```

结果为

```
Feily Zhang , 95.0
Haoyue Li , 100.0
Xin Chen , 94.0
```