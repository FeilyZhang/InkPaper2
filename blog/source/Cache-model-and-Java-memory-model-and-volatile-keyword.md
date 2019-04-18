title: "Cache模型与Java内存模型及volatile关键字"
date: 2018-12-22 19:54:06 +0800
update: 2018-12-22 19:54:06 +0800
author: me
cover: "-/images/java.jpg"
tags:
    - Java Core
preview: CPU比RAM的速度快得多，通常差距达上千倍，在极端情况下有上万倍的差距，那么为了协调RAM与CPU之间的速度矛盾提高CPU资源利用率，一般在CPU与RAM之间增设Cache，CPU访Cache的速度远高于访RAM的速度。

---

CPU比RAM的速度快得多，通常差距达上千倍，在极端情况下有上万倍的差距，那么为了协调RAM与CPU之间的速度矛盾提高CPU资源利用率，一般在CPU与RAM之间增设Cache，CPU访Cache的速度远高于访RAM的速度。

这实际上基于程序的局部性原理，在程序运行时，将运算所需要的数据从主内存中复制一份到CPU Cache中，这样CPU是直接对Cache进行数据的读出与写入的，当运算结束后，再将Cache中的数据刷回内存中。那么这样就会出现CPU缓存一致性问题。

## 一、CPU缓存一致性问题

CPU缓存一致性问题在单线程下是不存在的，因为对数据的操作始终是一个线程，但是在多线程高并发环境下就会出现线程不安全问题。因为每个线程都会有自己的工作内存，每次对内存中的数据操作时是把内存中的数据复制到自己工作内存中而不是直接对内存操作，以i++为例：

1. 假设存在记录访问数量的字段i的初始值为0；
2. 假定存在两个线程，分别对同一个i进行操作；
3. 由于对i的操作分为以下几步：
  1. 读取主内存的i到CPU Cache中；
  2. 对i进行+1操作；
  3. 将结果写回CPU Cache中；
  4. 最终将结果刷回RAM中。
4. 如果一个线程读取了i的值到自己的工作内存中还没来得及加1也就还没来得及刷回主内存，那么恰好丧失了对CPU的控制被调度到就绪队列，那么另一个线程获得对CPU的控制权再次读取i到自己的工作内存；
5. 那么两个线程分别加1都刷回主内存，最终结果仍然是1，但是我们的预期结果是2

这样就会造成线程不安全问题。究其缘由，造成线程不安全问题的原因是一线程对共享数据的修改对另一线程不可见。

举个例子，如下

```
import java.util.concurrent.TimeUnit;

public class VolatileFoo {
    final static int MAX = 5;
    static int init_value = 0;
    public static void main(String[] args) {
        new Thread(new Runnable() {
            @Override
            public void run() {
                int localValue = init_value;
                while (localValue < MAX) {
                    if (init_value != localValue) {
                        System.out.printf("The init_value is update to [%d]\n", init_value);
                        localValue = init_value;
                    }
                }
            }
            
        },"Reader").start();
        new Thread(new Runnable() {
            @Override
            public void run() {
                int localValue = init_value;
                while (localValue < MAX) {
                    System.out.printf("The init_value will be changed to [%d]\n" , ++localValue);
                    init_value = localValue;
                    try {
                        TimeUnit.MICROSECONDS.sleep(2);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
            
        },"Updater").start();
    }

}
```

Updater线程对init_value进行修改，Reader线程检测这种修改，一旦修改，那么就打印出来，结果呢？如下

```
The init_value will be changed to [1]
The init_value will be changed to [2]
The init_value will be changed to [3]
The init_value will be changed to [4]
The init_value will be changed to [5]
```

可以看到，修改不断进行，但是Reader线程并没有检测到这种修改，这就是init_value的修改对Reader不可见，什么原因呢？

Updater线程缓存init_value为0进入自己的工作内存，Reader也缓存init_value为0进入自己的工作内存，由于线程的工作内存都是独立的，那么Reader线程自然无法感知Updater线程对init_value的变动。

## 二、volatile关键字

如果我们给共享的变量加上volatile关键字，那么就能保证多线程间共享变量修改的可见性，如下

```
import java.util.concurrent.TimeUnit;

public class VolatileFoo {
    final static int MAX = 5;
    static volatile int init_value = 0;
    public static void main(String[] args) {
        new Thread(new Runnable() {
            @Override
            public void run() {
                int localValue = init_value;
                while (localValue < MAX) {
                    if (init_value != localValue) {
                        System.out.printf("The init_value is update to [%d]\n", init_value);
                        localValue = init_value;
                    }
                }
            }
            
        },"Reader").start();
        new Thread(new Runnable() {
            @Override
            public void run() {
                int localValue = init_value;
                while (localValue < MAX) {
                    System.out.printf("The init_value will be changed to [%d]\n" , ++localValue);
                    init_value = localValue;
                    try {
                        TimeUnit.MICROSECONDS.sleep(2);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
            
        },"Updater").start();
    }

}
```

结果为

```
The init_value will be changed to [1]
The init_value is update to [1]
The init_value will be changed to [2]
The init_value is update to [2]
The init_value will be changed to [3]
The init_value is update to [3]
The init_value will be changed to [4]
The init_value is update to [4]
The init_value will be changed to [5]
The init_value is update to [5]
```

volatile是怎样保证可见性的呢？这要了解一下Java内存模型

## 三、Java内存模型

Java内存模型(JMM)规定了Java虚拟机(JVM)如果与计算机的主存(RAM)进行工作,其决定了一个线程对共享变量的写入何时对其它线程可见，JMM定义的JVM与RAM之间的抽象关系，具体如下

1. 共享变量存储于主内存之中，每个线程都可以访问；
2. 每个线程都有私有的工作内存或者称为本地内存；
3. 工作内存只存储该线程对共享变量的副本；
4. 线程不能直接操作主内存，只有先操作了工作内存之后才能写回主内存；

举个例子，假设主内存的共享变量为0，线程1和线程2分别读取共享变量的副本进入自己的工作内存，假设线程1此时将共享变量修改为1同时刷回主内存中，当线程2想要去使用自己本地内存中共享变量的副本时就会发现该缓存的变量失效了，就不得不重新回到主存中重新缓存共享变量进自己的工作内存。

那么上述的例子就很好理解了，Updater和Reader线程分别缓存init_value进入自己的工作内存，Updater对init_value进行+1操作并立即刷回主内存中，同时Reader线程本地内存中的init_value失效，Reader重新缓存init_value，然后判断是否发生变化。

这就是volatile的作用，即保证共享变量对线程的可见性。

volatile与synchronized的区别如下

1. volatile只能作用于变量，synchronized作用在方法和代码块上；
2. volatile能保证可见性与有序性，但是无法保证原子性，synchronized不仅可以保证可见性还可以保证原子性；
3. volatile不会造成线程阻塞那么就不能用于同步线程，synchronized可以阻塞线程可以用于同步
4. volatile修饰的变量可以为null，synchronized关键字同步的monitor对象不能为null。