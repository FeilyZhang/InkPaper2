title: "七种单例模式的设计"
date: 2019-04-20 15:07:23 +0800
update: 2019-04-20 15:07:23 +0800
author: me
cover: "-/images/java.jpg"
tags:
    - Java Core
preview: 单例模式提供了一种在多线程情况下保证实例唯一性的解决方案，单例模式设计的标准是：懒加载、高性能、线程安全。

---

单例模式提供了一种在多线程情况下保证实例唯一性的解决方案，单例模式设计的标准是：懒加载、高性能、线程安全。

## 一、饿汉式

```
// final modifier, no inheritance allowed.
public final class Singleton {

    // Some business code.
    
    private static Singleton instance = new Singleton();
    
    // Private constructor, no external `new` is allowed.
    private Singleton() {
        
    }
    
    public static Singleton getInstance() {
        return instance;
    }
    
}
```

测试如下，我们启动10个线程测试其获得的线程实例

```
public class SingletonTest {

    public static void main(String[] args) {
        
        for (int i = 0; i < 10; i++) {
            new Thread(() -> {
                System.out.println(Thread.currentThread().getName() + " "
                    + Singleton.getInstance().toString());
            }, "thread-" + i).start();
        }
        
    }

}
```

输出为

```
thread-0 tech.feily.doc.thread.Singleton@df7eb94
thread-4 tech.feily.doc.thread.Singleton@df7eb94
thread-3 tech.feily.doc.thread.Singleton@df7eb94
thread-2 tech.feily.doc.thread.Singleton@df7eb94
thread-7 tech.feily.doc.thread.Singleton@df7eb94
thread-8 tech.feily.doc.thread.Singleton@df7eb94
thread-1 tech.feily.doc.thread.Singleton@df7eb94
thread-9 tech.feily.doc.thread.Singleton@df7eb94
thread-6 tech.feily.doc.thread.Singleton@df7eb94
thread-5 tech.feily.doc.thread.Singleton@df7eb94
```

可见，均为相同的实例。懒汉式的关键在于`instance`作为类变量并且直接得到了初始化，即如果主动使用(使用类变量属于主动使用，会导致类的加载并初始化)Singleton类，那么instance实例将会直接完成创建并初始化。具体而言，instance作为类变量在类初始化的过程中被收集进`<clinit>()`方法中，该方法能够百分百保证同步，即instance在多线程环境下不可能被实例化两次，但是instance被ClassLoader加载后可能很长一段时间才被使用，那么就意味着instance实例所开辟的堆内存会驻留更久的时间。

综上，这种办法的优点在于保证了多个线程下的唯一实例，在一个类中的成员属性比较少且占用的内存资源不多的情况下性能还可以，但是无法进行懒加载(即类实例在需要时再创建)。

## 二、懒汉式

懒汉式就是在使用类实例的时候再去创建，这样就避免了类在初始化时提前创建，代码如下

```
// final modifier, no inheritance allowed.
public final class Singleton {

    // Some business code.
    
    private static Singleton instance = null;
    
    // Private constructor, no external `new` is allowed.
    private Singleton() {
        
    }
    
    public static Singleton getInstance() {
        if (null == instance) {
            instance = new Singleton();
        }
        return instance;
    }
    
}
```

这样，`instance`在初始化阶段由于为`null`，那么就不会占用内存空间，在使用时即调用`getInstance()`方法后才会被实例化。

我们继续启动10个线程测试是否可以保证多线程环境下的实例唯一性，测试代码同上，输出为

```
thread-2 tech.feily.doc.thread.Singleton@59e410f2
thread-5 tech.feily.doc.thread.Singleton@38f983ba
thread-0 tech.feily.doc.thread.Singleton@38f983ba
thread-4 tech.feily.doc.thread.Singleton@38f983ba
thread-1 tech.feily.doc.thread.Singleton@38f983ba
thread-3 tech.feily.doc.thread.Singleton@59e410f2
thread-8 tech.feily.doc.thread.Singleton@38f983ba
thread-7 tech.feily.doc.thread.Singleton@38f983ba
thread-6 tech.feily.doc.thread.Singleton@38f983ba
thread-9 tech.feily.doc.thread.Singleton@38f983ba
```

可见，并没有保证实例唯一性，此处instance可以看作是多线程的共享变量，即多线程对instance进行操作，那么就有可能某线程在判断了`instance == null`之后CPU时间片到而进入就绪状态，待重新进入运行态时就执行了`new`操作，所以线程不安全，无法保证多线程环境下的实例唯一性。

解决的办法就是加锁，即给共享变量加锁。

## 三、懒汉式 + 同步方法

通过给懒汉式`getInstance()`方法进行同步，那么就会实现线程安全即多线程环境下实例的唯一性，如下

```
// final modifier, no inheritance allowed.
public final class Singleton {

    // Some business code.
    
    private static Singleton instance = null;
    
    // Private constructor, no external `new` is allowed.
    private Singleton() {
        
    }
    
    public static synchronized Singleton getInstance() {
        if (null == instance) {
            instance = new Singleton();
        }
        return instance;
    }
    
}
```

同样的测试代码，输出为

```
thread-3 tech.feily.doc.thread.Singleton@5dd54814
thread-2 tech.feily.doc.thread.Singleton@5dd54814
thread-0 tech.feily.doc.thread.Singleton@5dd54814
thread-6 tech.feily.doc.thread.Singleton@5dd54814
thread-5 tech.feily.doc.thread.Singleton@5dd54814
thread-9 tech.feily.doc.thread.Singleton@5dd54814
thread-1 tech.feily.doc.thread.Singleton@5dd54814
thread-4 tech.feily.doc.thread.Singleton@5dd54814
thread-7 tech.feily.doc.thread.Singleton@5dd54814
thread-8 tech.feily.doc.thread.Singleton@5dd54814
```

没毛病了。但是`synchronized`关键字又导致了`getInstance()`方法只能在同一时刻被一个线程访问，性能低下。