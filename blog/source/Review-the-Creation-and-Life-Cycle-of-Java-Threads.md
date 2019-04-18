title: "复习Java线程的创建与其生命周期"
date: 2018-12-18 19:08:53 +0800
update: 2018-12-18 19:08:53 +0800
author: me
cover: "-/images/java.jpg"
tags:
    - Java Core
preview: Java线程的两种创建方式与线程的生命周期。

---

## 一、Java线程的两种创建方式

### 1.1 继承Thread类

其步骤为继承Thread类并重写run方法，创建对象并调用start方法

```
public class HelloThread extends Thread {
    @Override
    public void run() {
        System.out.println(System.currentTimeMillis() + "," + Thread.currentThread().getName() + ":hello");
        System.out.println(System.currentTimeMillis() + "," + Thread.currentThread().getName() + Thread.currentThread().getState());
        System.out.println(System.currentTimeMillis() + "," + Thread.currentThread().getName() + Thread.currentThread().isAlive());
    }
    
    public static void main(String[] args) throws InterruptedException {
        for (int i = 0; i < 3; i++) {
            HelloThread thread = new HelloThread();
            thread.start();
        }
    }
}
```

输出为

```
1545129625507,Thread-2:hello
1545129625508,Thread-4:hello
1545129625509,Thread-4RUNNABLE
1545129625509,Thread-4true
1545129625508,Thread-2RUNNABLE
1545129625509,Thread-2true
1545129625507,Thread-3:hello
1545129625509,Thread-3RUNNABLE
1545129625509,Thread-3true
```

我们启动了三个线程，他们是并发执行的，可以看出，它们几乎是在同时执行的。执行顺序并不确定，线程接收CPU的调度。

### 1.2 实现Runnable接口

```
public class HelloThread implements Runnable {
    @Override
    public void run() {
        System.out.println(System.currentTimeMillis() + "," + Thread.currentThread().getName() + ":hello");
        System.out.println(System.currentTimeMillis() + "," + Thread.currentThread().getName() + Thread.currentThread().getState());
        System.out.println(System.currentTimeMillis() + "," + Thread.currentThread().getName() + Thread.currentThread().isAlive());
    }
    
    public static void main(String[] args) throws InterruptedException {
        for (int i = 0; i < 3; i++) {
            Thread thread = new Thread(new HelloThread());
            thread.start();
        }
    }
}
```

输出为

```
1545129841418,Thread-0:hello
1545129841418,Thread-2:hello
1545129841418,Thread-1:hello
1545129841418,Thread-2RUNNABLE
1545129841418,Thread-1RUNNABLE
1545129841418,Thread-0RUNNABLE
1545129841418,Thread-1true
1545129841418,Thread-2true
1545129841418,Thread-0true
```

## 二、Java线程的生命周期

![](https://cdn.nlark.com/yuque/0/2019/png/257195/1549874605826-e2982cd9-d892-4ef4-8534-559191202b0a.png#align=left&display=inline&height=244&name=%E4%B8%8B%E8%BD%BD.png&originHeight=316&originWidth=966&size=30900&width=746)

> 图片来源于：[https://www.cnblogs.com/sunddenly/p/4106562.html](https://www.cnblogs.com/sunddenly/p/4106562.html)

1. 新建状态：使用new关键字创建一个线程对象，那么就处于新建状态；
1. 就绪状态：
  1. 调用start方法进入就绪状态，在就绪队列中等待CPU调度；
  1. 当时间片到或者调用yield方法让出CPU使用权时重新进入就绪队列等待CPU调度；
1. 运行状态：运行状态当获得CPU调度，执行run方法时，就进入运行状态；
1. 阻塞状态：当调用sleep方法(睡眠时间到，进入就绪状态)、IO阻塞(线程调用的阻塞式IO方法已经返回,线程进入就绪状态)、等待同步锁(线程成功地获得了试图取得的同步监视器,进入就绪状态)、等待通知(线程正在等待某个通知时，其他线程发出了个通知，线程进入就绪状态)和suspend(处于挂起状态的线程被调甩了resdme()恢复方法,进入就绪状态)时进入阻塞状态；
1. 死亡状态：线程执行完毕(run()或call()方法执行完成)、调用stop方法(该方法容易导致死锁，通常不推荐使用)、线程抛出一个未捕获的Exception或Error,都会使线程死亡

需要注意的是以下方法已经过时，应该尽可能减少使用

```
public final void stop()
public final void suspend()
public final void resume()
```