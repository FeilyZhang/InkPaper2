title: "Java泛型浅析"
date: 2018-12-08 16:40:34 +0800
update: 2018-12-08 16:40:34 +0800
author: me
cover: "-/images/java.jpg"
tags:
    - Java Core
preview: 泛型，即广泛的类型，也就是不指定数据类型，而是在创建对象的时候在根据实际情况设置数据类型。

---

泛型，即广泛的类型，也就是不指定数据类型，而是在创建对象的时候在根据实际情况设置数据类型，一个简单的泛型类如下

```
public class Pair<T> {
	
    T first;
    T second;
	
    public Pair() {
		
    }
	
    public Pair(T first, T second) {
        this.first = first;
        this.second = second;
    }
	
    public void setFirst(T first) {
        this.first = first;
    }
	
    public void setSecond(T second) {
        this.second = second;
    }
	
    public T getFirst() {
        return first;
    }
    
    public T getSecond() {
        return second;
    }
	
}
```

我们用这个泛型类实现简单的JavaBean，那么其中的参数就可以在我们new对象时根据实际情况制定，可以是任意类型(包装类)，如下

```
public class Main {

    public static void main(String[] args) {
        Pair<String> pair = new Pair<String>();
        pair.setFirst("hello");
        pair.setSecond("world");
        System.out.println(pair.getFirst() + " " + pair.getSecond());
		
        Pair<Integer> pair2 = new Pair<Integer>();
        pair2.setFirst(100);
        pair2.setSecond(200);
        System.out.println(pair2.getFirst() + pair2.getSecond());
    }
}
```

输出为

```
hello world
300
```

同样的功能也可以使用Object类型实现，如下

```
public class Pair1 {

    Object first;
    Object second;
    
    public Pair1() {
        
    }
    
    public Pair1(Object first, Object second) {
        this.first = first;
        this.second = second;
    }
    
    public void setFirst(Object first) {
        this.first = first;
    }
    
    public void setSecond(Object second) {
        this.second = second;
    }
    
    public Object getFirst() {
        return first;
    }
    
    public Object getSecond() {
        return second;
    }
    
}
```

```
public class Pair1Main {

    public static void main(String[] args) {
        Pair1 pair1 = new Pair1();
        pair1.setFirst("hello");
        pair1.setSecond("world");
        System.out.println((String)pair1.getFirst() + " " + (String)pair1.getSecond());
        Pair1 pair2 = new Pair1();
        pair2.setFirst(100);
        pair2.setSecond(200);
        System.out.println((Integer)pair2.getFirst() + (Integer)pair2.getSecond());
    }

}
```

与泛型实现有着同样的输出

这两种方式有什么区别呢？

对于泛型类，Java编译器会将泛型代码转换为普通的非泛型代码，这种转换叫做**擦除**，即将类型参数T擦除，替换为Object并插入必要的强制类型转换，也就是说泛型类被擦除之后会变成上述带有Object的代码。

这样做有什么好处呢？既然这么麻烦为什么不直接用Object呢？

泛型的好处为：**更好的安全性和更好的可读性**。

如果用Object实现可变类型的话，那么如果在代码编写过程中，开发环境和编译器并不能发现错误就会导致bug，但是如果用泛型的话开发环境和编译器会直接给我们报错，安全性更强；而且通过使用泛型，可以省去强制类型转换，方便了很多，可读性明显提升。
