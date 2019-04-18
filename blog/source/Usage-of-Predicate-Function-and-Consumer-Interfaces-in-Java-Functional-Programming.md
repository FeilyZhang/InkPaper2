title: "Java函数式编程之Predicate、Function、Consumer接口的用法"
date: 2018-12-11 19:19:45 +0800
update: 2018-12-11 19:19:45 +0800
author: me
cover: "-/images/java.jpg"
tags:
    - Java Core
preview: Predicate、Function、Consumer接口的用法。

---

## 一、Predicate接口及其test方法的签名如下

```
Predicate<T>
boolean test<T t>
```

即泛型T为test形参的数据类型，也就是test判断数据类型为T的形参是否满足条件

## 二、Function接口及其apply方法的签名如下

```
Function<T, R>
R apply(T t)
```

即Function接口共有两种泛型，分别是T和R，从apply方法来看其中T为输入的数据类型，R为输出的数据类型，也就是apply方法将操作施加在类型为T的参数身上然后返回类型为R的返回值。

## 三、Consumer接口及其accept方法的签名如下

```
Comsumer<T>
void accept(T t)
```

即Cosumer接口只有一种类型，为T，accept方法是直接对原对象进行操作，这些都是在编写代码时要注意的。

## 四、示例

用一个例子说明之，也是上文的一个例子，只不过综合运用了这三种函数接口

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

```
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;
import java.util.function.Consumer;
import java.util.function.Function;
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

        List<String> resultMap = map(list, student -> student.getScore() > 90 ? student.getName() : null);
        for (String e : resultMap) {
            if (e != null) {
                System.out.println(e);
            }
        }

        forEach(list, student -> student.setName(student.getName().toLowerCase()));
        for (Student e : list) System.out.println(e.getName());
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
    
    public static <T, R> List<R> map (List<T> list, Function<T, R> mapper) {
        List<R> reList = new ArrayList<R>(list.size());
        for (T e : list) {
            reList.add(mapper.apply(e));
        }
        return reList;
    }
    
    public static <T> void forEach(List<T> list, Consumer<T> consumer) {
        for (T e : list) {
            consumer.accept(e);
        }
    }
}
```

有如下输出

```
Feily Zhang , 95.0
Haoyue Li , 100.0
Xin Chen , 94.0
Feily Zhang
Haoyue Li
Xin Chen
feily zhang
haoyue li
daji luobu
xin chen
```