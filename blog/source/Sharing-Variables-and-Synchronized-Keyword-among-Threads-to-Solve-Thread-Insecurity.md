title: "线程间共享变量与synchronized关键字解决线程不安全问题"
date: 2018-12-19 15:47:07 +0800
update: 2018-12-19 15:47:07 +0800
author: me
cover: "-/images/java.jpg"
tags:
    - Java Core
preview: 由于线程有自己的程序计数器、局部变量表和操作数栈，那么也就意味着每实例化一个继承了Thread的类或者实现了Runnable接口的类，就意味着创建了一个对象，这些都是线程私有的，也就无法对变量实现共享。

---

由于线程有自己的程序计数器、局部变量表和操作数栈，那么也就意味着每实例化一个继承了Thread的类或者实现了Runnable接口的类，就意味着创建了一个对象，这些都是线程私有的，也就无法对变量实现共享。

想共享变量，即变量在多个实例访问的情况下仍然是唯一的(指的是只有这一个变量，不会因new关键字而产生对象的变量)，在多线程并发的情况下有两种方式实现：

+ 第一种是，使用类变量。这种变量存在于方法区中，由线程共享，每个方法或者线程都拥有对该线程的访问权，那么就能够实现变量共享的需求。
+ 第二种是，让线程使用同一个Runnable接口，这样他们的资源仍然是共享的。

如下，示例一个叫号机程序，每天放号50个，共有四台出号机，也就意味着共有四个线程，即四个线程对共享变量进行操作，该共享变量就是号数，小于等于50，先用第一种方式来实现

由于50比较大，不好输出结果，不妨改为10

```
public class TicketWindow extends Thread {
    private final String name;
    private static int num = 1;
    private static final int maxNum = 10;
    public TicketWindow(String name) {
        this.name = name;
    }
    @Override
    public void run() {
        while (num <= maxNum) {
            System.out.println("Window:" + name + " current number is " + (num++));
        }
    }
}

public class TicketWindowMain {

    public static void main(String[] args) {
        TicketWindow tw1 = new TicketWindow("First");
        TicketWindow tw2 = new TicketWindow("Second");
        TicketWindow tw3 = new TicketWindow("Third");
        tw1.start();
        tw2.start();
        tw3.start();
        
    }

}
```

结果为

```
Window:Second current number is 1
Window:Third current number is 2
Window:First current number is 1
Window:Third current number is 4
Window:Second current number is 3
Window:Third current number is 6
Window:First current number is 5
Window:Third current number is 8
Window:Second current number is 7
Window:Third current number is 10
Window:First current number is 9
```

事实上，每次执行的结果是不一样的，可以看到1被抽了两次，这是因为变量num的访问不是原子的，对num的操作分为两步，第一步是先读num，然后再加1；当Second线程读取了num之后还没来得及加1就丧失了CPU的使用权，被First线程获取到了CPU使用权，因为First仍然拿到了1，此时正常加1，但是为什么2会比1输出早呢？因为加1之后才会输出，First线程加1之后又丧失了CPU使用权，被调度到Third线程，Third线程完成了所有步骤。

可见，此方式虽然实现了对共享变量的访问，但是对共享变量的操作不是原子的，存在线程安全问题(即一个线程状态对另一个线程会产生影响)。

可见，线程不安全的问题主要产生在共享变量num身上，即对其操作不原子，如果我们用synchronized关键字修饰涉及对共享变量操作的代码块或者方法，那么就相当于锁住了共享变量，在一个线程访问共享变量开始就对共享变量加锁，一直到对共享变量的访问结束(而不是线程结束)才会释放锁，期间其它线程想要获得锁只能等待，处于阻塞状态，当该线程释放锁，那么下一个排队的线程就会拿到锁而进入就绪状态等待CPU调度。

还有第二种方法，即多个线程使用相同的Runnable接口，如下

```
public class TicketWindow implements Runnable {
    private int num = 1;
    private final int maxNum = 10;
    @Override
    public void run() {
        while (num <= maxNum) {
            System.out.println(Thread.currentThread() + " current number is " + (num++));
        }
    }
}

public class TicketWindowMain {

    public static void main(String[] args) {
        TicketWindow tw = new TicketWindow();
        Thread t1 = new Thread(tw, "First");
        Thread t2 = new Thread(tw, "Second");
        Thread t3 = new Thread(tw, "Third");
        t1.start();
        t2.start();
        t3.start();
        
    }

}
```

结果为

```
Thread[Second,5,main] current number is 2
Thread[First,5,main] current number is 2
Thread[Third,5,main] current number is 1
Thread[First,5,main] current number is 4
Thread[Second,5,main] current number is 3
Thread[First,5,main] current number is 6
Thread[Third,5,main] current number is 5
Thread[First,5,main] current number is 8
Thread[Second,5,main] current number is 7
Thread[First,5,main] current number is 10
Thread[Third,5,main] current number is 9
```

和第一种方法一样，虽然实现了对共享变量的访问，但是还是线程不安全，明明十次但是却有十一个输出，总之会产生各种各样的问题。对关键代码块加`synchronized`关键字即可解决问题

```
public class TicketWindow implements Runnable {
    private int num = 1;
    private static final int maxNum = 10;
    private static final Object MUTEX = new Object();
    @Override
    public void run() {
        while (num <= maxNum) {
            synchronized(MUTEX) {
                System.out.println(Thread.currentThread() + " current number is " + (num++));
            }
        }
    }
}
```

主文件不变，结果为

```
Thread[First,5,main] current number is 1
Thread[First,5,main] current number is 2
Thread[First,5,main] current number is 3
Thread[First,5,main] current number is 4
Thread[First,5,main] current number is 5
Thread[Third,5,main] current number is 6
Thread[Third,5,main] current number is 7
Thread[Third,5,main] current number is 8
Thread[Third,5,main] current number is 9
Thread[Third,5,main] current number is 10
Thread[Second,5,main] current number is 11
Thread[First,5,main] current number is 12
```

**synchronized关键字的粒度应该尽可能小**，因为粒度越大效率越低越不能发挥出多线程并发的优势，因为对非共享资源加锁本身就是资源的浪费。比如锁加在while循环外面的话，那么就会出现一个线程从头到尾的执行的现象，这是因为，锁中的代码执行完毕之后该线程才会释放锁其余线程才有机会拿到锁，锁中是while循环，那么也就意味着while循环执行完毕才会释放锁，由于三个线程用一个实现了Runnable接口的类，所以共享变量达到要求之后就会线程终止，那么其余线程就没机会执行。

但是不是应该输出到10吗，怎么到了12？这可能是因为Third线程再获得锁之后(修改num之前)丧失了CPU使用权，此后Second线程与First线程看到自身条件仍然满足(10 <= 10),只是拿不到锁无法进一步操作，但是等Third线程操作完成释放锁之后他们就继续操作，因为他们已经判断过了while条件，他们的程序计数器记得，所以直接拿到锁输出。

这就简单了，让他们拿到锁之后再判断一次，重新看一下num的值是否符合条件，如下

```
public class TicketWindow implements Runnable {
    private int num = 1;
    private static final int maxNum = 10;
    private static final Object MUTEX = new Object();
    @Override
    public void run() {
        while (num <= maxNum) {
            synchronized(MUTEX) { 
                if (num <= maxNum) { 
                    System.out.println(Thread.currentThread() + " current number is " + (num++)); 
                } 
            } 
        } 
    } 
}
```

输出为

```
Window:First current number is 1
Window:Third current number is 2
Window:Third current number is 3
Window:Third current number is 4
Window:Third current number is 5
Window:Third current number is 6
Window:Third current number is 7
Window:Third current number is 8
Window:Third current number is 9
Window:Third current number is 10
```

问题解决！

两种方法的区别在于static修饰的变量和方法随类的生存而生存，生命周期很长，所以尽可能使用实现了Runnable接口的办法。

----------2018-12-21 17：29 更新----------

还有一种办法就是使用相同的类实例，让多线程调用相同的类实例方法来实现，如下(没加锁，需要注意)

```
public class TicketWindow {
    private int num = 1;
    private final int maxNum = 10;
    public void run() {
        while (num <= maxNum) {
            System.out.println(Thread.currentThread() + " current number is " + (num++));
        }
    }
}
public class TicketWindowMain {

    public static void main(String[] args) {
        TicketWindow tw = new TicketWindow();
        Thread t1 = new Thread(new Runnable() {
            @Override
            public void run() {
                tw.run();
            }
            
        }, "First");
        Thread t2 = new Thread(new Runnable() {
            @Override
            public void run() {
                tw.run();
            }
            
        },"Second");
        Thread t3 = new Thread(new Runnable() {
            @Override
            public void run() {
                tw.run();
            }
            
        },"Third");
        t1.start();
        t2.start();
        t3.start();
        
    }

}
```

结果为

```
Thread[Second,5,main] current number is 2
Thread[Third,5,main] current number is 3
Thread[First,5,main] current number is 1
Thread[Third,5,main] current number is 5
Thread[Second,5,main] current number is 4
Thread[Third,5,main] current number is 7
Thread[First,5,main] current number is 6
Thread[Third,5,main] current number is 9
Thread[Second,5,main] current number is 8
Thread[First,5,main] current number is 10
```