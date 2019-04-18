title: "深入理解Thread类构造函数"
date: 2018-12-19 16:29:13 +0800
update: 2018-12-19 16:29:13 +0800
author: me
cover: "-/images/java.jpg"
tags:
    - Java Core
preview: 由于线程有自己的程序计数器、局部变量表和操作数栈，那么也就意味着每实例化一个继承了Thread的类或者实现了Runnable接口的类，就意味着创建了一个对象，这些都是线程私有的，也就无法对变量实现共享。

---

## 一、线程的名称

线程可以通过当前线程的getName()方法获取该线程在运行期间唯一的名称，如下

```
public class TestMain {

    public static void main(String[] args) {
        System.out.println(Thread.currentThread().getName());
        for (int i = 0; i < 3; i++) {
            Thread thread = new Thread(new Runnable() {
                @Override
                public void run() {
                    System.out.println(Thread.currentThread().getName());
                }
            });
            thread.start();
        }
    }
}
```

我们启动了三个线程，线程的逻辑写在run方法之内，目的是输出线程name，输出为

```
main
Thread-0
Thread-1
Thread-2
```

getId()方法也可以获得线程的Id，与getName使用方法一样

## 二、线程的命名

我们也可以自定义线程的名称，这便于我们监控线程的执行状态和排错，方法就是在Thread类的第二个参数设置名称，在run方法内通过Thread.currentThread()方法来获取，如下

```
public class TestMain {

    public static void main(String[] args) {
        System.out.println(Thread.currentThread().getName());
        for (int i = 0; i < 3; i++) {
            Thread thread = new Thread(new Runnable() {
                @Override
                public void run() {
                    System.out.println(Thread.currentThread());
                }
            },"My Thead-" + i);
            thread.start();
        }
    }
}
```

输出为

```
main
Thread[My Thead-0,5,main]
Thread[My Thead-1,5,main]
Thread[My Thead-2,5,main]
```

该构造方法的原型为

```
Thread(Runnable target, String name)
```

## 三、线程的重新命名

线程在run之前，还可以修改线程名称，即new之后run之前还可以修改，方式为Thread.setName()方法，这是最后一次修改机会，线程在运行时是不能修改名称的，如下

```
public class TestMain {

    public static void main(String[] args) {
        System.out.println(Thread.currentThread().getName());
        for (int i = 0; i < 3; i++) {
            Thread thread = new Thread(new Runnable() {
                @Override
                public void run() {
                    System.out.println(Thread.currentThread());
                }
            },"My Thead-" + i);
            //thread.setName("Your Thread+" + i);
            thread.start();
            thread.setName("Your Thread+" + i);
        }
    }
}
```

输出为

```
main
Thread[Your Thread+0,5,main]
Thread[Your Thread+1,5,main]
Thread[Your Thread+2,5,main]
```

## 四、Thread与ThreadGroup

在Thread的构造函数中，可以显式的指定线程的Group，如果没有显式的指定Group那么该线程就会被假如父线程的Group，Thread设置Group的构造方法为

```
Thread(ThreadGroup group, String name)
Thread(ThreadGroup group, Runnable target, String name)
Thread(ThreadGroup group, Runnable target, String name, long stackSize)
```

ThreadGroup的构造方法为

```
ThreadGroup group = new ThreadGroup(String groupName)
```

测试一下

```
public class TestMain {

    public static void main(String[] args) {
        Thread t1 = new Thread("t1");
        ThreadGroup group = new ThreadGroup("My Group");
        Thread t2 = new Thread(group, "t2");
        System.out.println(t1.getThreadGroup());    //java.lang.ThreadGroup[name=main,maxpri=10]
        System.out.println(t2.getThreadGroup());    //java.lang.ThreadGroup[name=My Group,maxpri=10]
        System.out.println(t1.getThreadGroup() == group);   //false
        System.out.println(t2.getThreadGroup() == group);   //true
        System.out.println(Thread.currentThread().getThreadGroup() == group);   //false
        
    }
}
```

## 五、Thread与Runnable的区别

二者看似都可以创建线程，但是二者其实是有区别的，区别就是Thread负责线程本身相关职责和控制，Runnable负责线程逻辑执行单元部分。上述例子已经很好的说明了这一点。