title: "Thread API详细介绍"
date: 2018-12-19 20:55:22 +0800
update: 2018-12-19 20:55:22 +0800
author: me
cover: "-/images/java.jpg"
tags:
    - Java Core
preview: sleep()、yield()、设置线程的优先级、线程interrupt、join()方法以及线程的关闭。

---

## 一、`sleep()`方法

sleep()方法的签名如下

```
public static void sleep(long millis) throws InterruptedException
public static void sleep(long millis, int nanos) throws InterruptedException
```

该方法会使当前线程睡眠millis毫秒后重新进入就绪状态参与CPU竞争，但是需要注意的是该方法即时休眠也不会放弃monitor锁(前提是获得该锁)，示例代码如下

```
public class TestMain {
    public static int shareVar = 10;
    public static final Object MUTEX = new Object();
    public static void main(String[] args) {
        for (int i = 0; i < 3; i++) {
            new Thread(new Runnable() {
                @Override
                public void run() {
                    System.out.println(Thread.currentThread().getName() + ": I will sleep");
                    synchronized(MUTEX) {
                        try {
                            Thread.sleep(5000);
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                        System.out.println(Thread.currentThread().getName() + ": " + (shareVar++));
                    }
                    System.out.println(Thread.currentThread().getName() + ": Finish");
                } 
            }).start();
        }
    }
}
```

输出为

```
Thread-0: I will sleep
Thread-2: I will sleep
Thread-1: I will sleep
Thread-0: 10
Thread-0: Finish
Thread-1: 11
Thread-1: Finish
Thread-2: 12
Thread-2: Finish
```

我们启动了3个线程，每个线程在进入同步代码块后休眠5s，从输出可以看出，线程Thread-0首先获得了调度并首先进入同步代码块，然后休眠。由于获得锁然后休眠的线程是不释放锁的，也就是说一直退出同步代码块之后才会释放锁，那么我们可以确定一定是Thread-0先打印输出10

需要注意的是一定是获得共享变量所在锁之后睡眠的话才不会释放锁，如果在获得锁之前或者释放锁之后睡眠，那么是不会独占锁的(锁存在的意义是保证数据的一致性),那么对共享变量的操作不会因为谁先睡眠而先操作，还是以CPU调度下哪个线程先进入同步代码块获得锁为准。

示例如下，第一个线程在获得锁之前就睡眠，那么待苏醒后首先对共享变量操作的不一定是该线程

```
public class TestMain {
    public static int shareVar = 10;
    public static final Object MUTEX = new Object();
    public static void main(String[] args) {
        for (int i = 0; i < 3; i++) {
            new Thread(new Runnable() {
                @Override
                public void run() {
                    System.out.println(Thread.currentThread().getName() + ": I will sleep");
                    try {
                        Thread.sleep(5000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    synchronized(MUTEX) {
                        System.out.println(Thread.currentThread().getName() + ": " + (shareVar++));
                    }
                    System.out.println(Thread.currentThread().getName() + ": Finish");
                } 
            }).start();
        }
    }
}
```

输出为

```
Thread-0: I will sleep
Thread-2: I will sleep
Thread-1: I will sleep
Thread-2: 10
Thread-2: Finish
Thread-1: 11
Thread-1: Finish
Thread-0: 12
Thread-0: Finish
```

显而易见！

所以想睡眠独占锁，必须先进入锁再睡眠！

## 二、`yield()`方法

该方法会自动让出对CPU的使用权，但是CPU不一定受理，即不一定有作用，如果CPU资源不紧张，那么就会忽略这种提醒。

由于是让出CPU使用权，所以该方法的调用会使当前线程从RUNNING状态(运行状态)切换到RUNNABLE状态(就绪状态)。

## 三、`yield()`方法与`sleep()`方法的比较

在JDK1.5之前的版本，yield()方法实际上是调用了sleep(0)方法，即让当前线程休眠0毫秒直接进入就绪队列，那么很自然地就是将CPU控制权交给了下一个线程，但是二者其实存在着本质的差别：

+ `sleep()`方法会导致当前线程暂停指定时间，没有CPU时间片的消耗；
+ `yield()`方法只是对CPU调度器的一个提示，如果CPU调度器没有忽略这个提示，那么就会导致线程上下文的切换(即切换到下一个线程，本线程让出CPU控制权)；
+ `sleep()`会导致线程短暂的block，会在给定时间内释放CPU资源；
+ `yield()`会使线程由运行状态进入就绪状态，前提是CPU没有忽略这个提示；
+ `sleep()`方法几乎百分百完成了给定时间的休眠，但是`yield()`则不一定；
+ 一个线程`sleep()`，另一个线程调用`interrupt()`会捕捉到中断信号，而`yield()`不会。

## 四、设置线程的优先级

```
public final void setPrioity(int newPrioity); //设置线程优先级
public final int getPrioity() //获取线程优先级
```

线程优先级是一个1-10的int型数字，默认为5，数字越大优先级越高，但是因平台的差异性，优先级不可过分依赖，平台可能会忽视优先级。

```
public class TestMain {
    public static int shareVar = 10;
    public static final Object MUTEX = new Object();
    public static void main(String[] args) {
        Thread t1 = new Thread(new Runnable() {
            @Override
            public void run() {
                int i = 0;
                while (i < 5) {
                    System.out.println(Thread.currentThread());
                    i++;
                }
            }
        }, "t1");
        Thread t2 = new Thread(new Runnable() {
            @Override
            public void run() {
                int i = 0;
                while (i < 5) {
                    System.out.println(Thread.currentThread());
                    i++;
                }
            }
        }, "t2");
        t1.setPriority(1);
        t2.setPriority(10);
        t1.start();
        t2.start();
    }
}
```

以上代码设置t1线程的优先级为1，t2的优先级为10，按理说，t2的输出次数应该高于t1，但是实际情况并不是这样，可能就是平台忽视了优先级，我的平台为win8

## 五、线程interrupt

共有以下三个方法

```
public void interrupt()
public static boolean interrupted()
public boolean isInterrupted()
```

interrupt()方法可以打断以下方法调用引起的线程阻塞

```
Object的wait方法；
Object的wait(long)方法；
Object的wait(long, int)方法；
Thread的sleep(long)方法；
Thread的sleep(long, int)方法；
Thread的join方法；
Thread的join(long)方法；
Thread的join(long, int)方法；
InterruptibleChannel的io操作；
Selector的wakeup方法
其它方法。
```

上述方法的执行都会使线程进入阻塞状态，若另外一个线程调用被阻塞线程的interrupt方法时就会打断这种阻塞。一个线程在阻塞情况下被打断都会抛出一个TnterruptedException的异常，这个异常就像一个信号一样通知当前线程被打断了。

示例如下

```
import java.util.concurrent.TimeUnit;

public class TestMain {
    public static int shareVar = 10;
    public static final Object MUTEX = new Object();
    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(new Runnable() {
            @Override
            public void run() {
                long start = System.currentTimeMillis();
                System.out.println("SubThread : I will sleep 1 minutes");
                try {
                    TimeUnit.MINUTES.sleep(1);
                } catch (InterruptedException e) {
                    System.out.println("oh, I am be interrupted");
                }
                if (System.currentTimeMillis() - start <= 60000) {
                    System.out.println("SubThread : I wake up, oh, I didn't sleep enough. I was sleepy.");
                } else {
                    System.out.println("SubThread : I wake up, oh, I've had enough sleep.");
                }
            }
        }, "t1");
        t1.start();
        TimeUnit.MICROSECONDS.sleep(2);
        t1.interrupt();
    }
}
```

线程t1在运行时打算睡眠1分钟，在t1启动后，先将主线程休眠2毫秒，这是为了让t1线程处于就绪状态，然后主线程调用t1的interrupt方法打断睡眠，强制叫醒了t1线程，输出为

```
SubThread : I will sleep 1 minutes
oh, I am be interrupted
SubThread : I wake up, oh, I didn't sleep enough. I was sleepy.
```

`isInterrupted`用于判断当前线程是否被中断，那我们就可以如下改造上述程序

```
import java.util.concurrent.TimeUnit;

public class TestMain {
    public static int shareVar = 10;
    public static final Object MUTEX = new Object();
    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(new Runnable() {
            @Override
            public void run() {
                long start = System.currentTimeMillis();
                System.out.println("SubThread : I will sleep 1 minutes");
                try {
                    TimeUnit.MINUTES.sleep(1);
                } catch (InterruptedException e) {
                    System.out.println("oh, I am be interrupted");
                }
                if (System.currentTimeMillis() - start <= 60000) {
                    System.out.println("SubThread : I wake up, oh, I didn't sleep enough. I was sleepy.");
                } else {
                    System.out.println("SubThread : I wake up, oh, I've had enough sleep.");
                }
            }
        }, "t1");
        t1.start();
        TimeUnit.MICROSECONDS.sleep(2);
        if (!t1.isInterrupted()) {
            t1.interrupt();
        }
    }
}
```

有同样的输出，主线程判断t1线程时候被中断，如果被中断则什么也不干，如果没有则中断它，把它从睡眠中叫醒进入进入就绪状态，实际上sleep会捕获到interrupt的中断信号，从而就会擦除interrupt标识，什么意思呢？就是一个线程被interrupt中断一次，那么其isInterrupted为true，如果被sleep捕获的话(即该线程中有sleep方法被interrupt中断)，sleep被苏醒后就会擦除掉isInterrupted标识，即为false，这样就能保证下次能够合理的中断(控制)，演示如下

```
import java.util.concurrent.TimeUnit;

public class TestMain {
    public static int shareVar = 10;
    public static final Object MUTEX = new Object();
    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(new Runnable() {
            @Override
            public void run() {
                long start = System.currentTimeMillis();
                System.out.println("SubThread : I will sleep 1 minutes");
                try {
                    TimeUnit.MINUTES.sleep(1);
                } catch (InterruptedException e) {
                    System.out.println("oh, I am be interrupted");
                }
                if (System.currentTimeMillis() - start <= 60000) {
                    System.out.println("SubThread : I wake up, oh, I didn't sleep enough. I was sleepy.");
                } else {
                    System.out.println("SubThread : I wake up, oh, I've had enough sleep.");
                }
            }
        }, "t1");
        t1.start();
        System.out.println(t1.isInterrupted());
        TimeUnit.MICROSECONDS.sleep(2);
        t1.interrupt();
        System.out.println(t1.isInterrupted());
        System.out.println(t1.isInterrupted());
    }
}
```

输出为

```
false
SubThread : I will sleep 1 minutes
false
oh, I am be interrupted
SubThread : I wake up, oh, I didn't sleep enough. I was sleepy.
false
```

在`t1.interrupt()`方法执行之前输出为false可以理解，在该方法执行之后还输出false，这是因为在sleep擦除掉了该标识。当然了，如果没有sleep方法在t1线程中，直接中断也就意味着该标识不会被擦除，演示如下

```
import java.util.concurrent.TimeUnit;

public class TestMain {
    public static int shareVar = 10;
    public static final Object MUTEX = new Object();
    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(new Runnable() {
            @Override
            public void run() {
                while (true) {
                    //System.out.println("I am t1 Thread, " + isInterrupted());
                }
            }
        }, "t1");
        t1.setDaemon(true);
        t1.start();
        TimeUnit.MICROSECONDS.sleep(2);
        System.out.println("Thread is interrupted? " + t1.isInterrupted());
        t1.interrupt();
        System.out.println("Thread is interrupted? " + t1.isInterrupted());
        System.out.println("Thread is interrupted? " + t1.isInterrupted());
    }
}
```

输出为

```
Thread is interrupted? false
Thread is interrupted? true
Thread is interrupted? true
```

可以看出，中断标识并没有被擦除。

`interrupted()`方法是一个静态方法，也就意味着可以在run方法中通过Thread调用，与isInterrupted()不同的是，isInterrupted()方法会因是否有sleep方法而被是否擦除标识，interrupted方法在run中调用会直接擦除标识，演示如下

```
import java.util.concurrent.TimeUnit;

public class TestMain {
    public static int shareVar = 10;
    public static final Object MUTEX = new Object();
    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(new Runnable() {
            @Override
            public void run() {
                while (true) {
                    System.out.println(Thread.interrupted());
                }
            }
        }, "t1");
        t1.setDaemon(true);
        t1.start();
        TimeUnit.MICROSECONDS.sleep(2);
        t1.interrupt();
    }
}
```

输出为

```
...
false
false
false
true
false
false
...
```

即很多次输出中只有一个true，这个true就是被主线程调用t1的interrupt方法之后输出的，但随后被Thread.interrupted()方法直接擦除掉了标识，所以输出为false

## 六、join()方法

join与sleep一样，会被interrupt中断，同样也会擦除interrupt标识。join某个线程之后，当前线程会进入阻塞状态(比如main线程执行t1.join()方法，那么main线程进入阻塞状态，直至t1线程执行完毕或者被其余线程中断或者时间到main线程才会进入就绪状态)，测试如下

```
import java.util.concurrent.TimeUnit;

public class TestMain {
    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(new Runnable() {
            int i = 0;
            @Override
            public void run() {
                while (i < 5) {
                    System.out.println(Thread.currentThread() + " : " + (i++));
                }
            }
        }, "t1");
        Thread t2 = new Thread(new Runnable() {
            int i = 0;
            @Override
            public void run() {
                try {
                    t1.join();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                while (i < 5) {
                    System.out.println(Thread.currentThread() + " : " + (i++));
                }
            }
        }, "t2");
        t1.start();
        t2.start();
        //t1.join();
        //TimeUnit.MICROSECONDS.sleep(2);
        //t1.interrupt();
    }
}
```

输出为

```
Thread[t1,5,main] : 0
Thread[t1,5,main] : 1
Thread[t1,5,main] : 2
Thread[t1,5,main] : 3
Thread[t1,5,main] : 4
Thread[t2,5,main] : 0
Thread[t2,5,main] : 1
Thread[t2,5,main] : 2
Thread[t2,5,main] : 3
Thread[t2,5,main] : 4
```

很明显，t2调用`t1.join()`阻塞了自己一直等到t1执行完毕(结束完整声明周期)才执行，如果给join方法加上时间那么输出就又不一样

```
import java.util.concurrent.TimeUnit;

public class TestMain {
    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(new Runnable() {
            int i = 0;
            @Override
            public void run() {
                while (i < 5) {
                    System.out.println(Thread.currentThread() + " : " + (i++));
                }
            }
        }, "t1");
        Thread t2 = new Thread(new Runnable() {
            int i = 0;
            @Override
            public void run() {
                try {
                    t1.join();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                while (i < 5) {
                    System.out.println(Thread.currentThread() + " : " + (i++));
                }
            }
        }, "t2");
        t1.start();
        t2.start();
        //t1.join();
        //TimeUnit.MICROSECONDS.sleep(2);
        //t1.interrupt();
    }
}
```

我们让t2阻塞，t1在没有t2竞争CPU的情况下独自占用CPU运行，由于只有1毫秒，5太小看不到预期结果，改为1000，那么运行就会发现1毫秒后t1与t2抢占式交替执行，这是因为t1对CPU的独占在1毫秒后就失效了。

再看这个

```
import java.util.concurrent.TimeUnit;

public class TestMain {
    public static void main(String[] args) throws InterruptedException {
        Thread t3 = new Thread(new Runnable() {
            int i = 0;
            @Override
            public void run() {
                while (i < 5) {
                    System.out.println(Thread.currentThread() + " : " + (i++));
                }
            }
        }, "t3");
        Thread t1 = new Thread(new Runnable() {
            int i = 0;
            @Override
            public void run() {
                try {
                    t3.join();
                } catch (InterruptedException e) {
                    // TODO Auto-generated catch block
                    e.printStackTrace();
                }
                while (i < 5) {
                    System.out.println(Thread.currentThread() + " : " + (i++));
                }
            }
        }, "t1");
        Thread t2 = new Thread(new Runnable() {
            int i = 0;
            @Override
            public void run() {
                try {
                    t1.join();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                while (i < 5) {
                    System.out.println(Thread.currentThread() + " : " + (i++));
                }
            }
        }, "t2");
        t1.start();
        t2.start();
        t3.start();
        //t1.join();
        //TimeUnit.MICROSECONDS.sleep(2);
        //t1.interrupt();
    }
}
```

t2把CPU使用权让渡给t1，t1让渡给t3，那么执行顺序就是t3>t1>t2，如下

```
Thread[t3,5,main] : 0
Thread[t3,5,main] : 1
Thread[t3,5,main] : 2
Thread[t3,5,main] : 3
Thread[t3,5,main] : 4
Thread[t1,5,main] : 0
Thread[t1,5,main] : 1
Thread[t1,5,main] : 2
Thread[t1,5,main] : 3
Thread[t1,5,main] : 4
Thread[t2,5,main] : 0
Thread[t2,5,main] : 1
Thread[t2,5,main] : 2
Thread[t2,5,main] : 3
Thread[t2,5,main] : 4
```

我们再改动一下上面的程序，t2将权力让渡给t3，我们让t1调用t3的interrupt方法中断t3，会出现什么情况，如下

```
import java.util.concurrent.TimeUnit;

public class TestMain {
    public static void main(String[] args) throws InterruptedException {
        Thread t3 = new Thread(new Runnable() {
            int i = 0;
            @Override
            public void run() {
                while (i < 5) {
                    System.out.println(Thread.currentThread() + " : " + (i++) + " " + Thread.interrupted());
                }
            }
        }, "t3");
        Thread t1 = new Thread(new Runnable() {
            int i = 0;
            @Override
            public void run() {
                t3.interrupt();
                while (i < 5) {
                    System.out.println(Thread.currentThread() + " : " + (i++));
                }
            }
        }, "t1");
        Thread t2 = new Thread(new Runnable() {
            int i = 0;
            @Override
            public void run() {
                try {
                    t3.join();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                while (i < 5) {
                    System.out.println(Thread.currentThread() + " : " + (i++));
                }
            }
        }, "t2");
        t1.start();
        t2.start();
        t3.start();
    }
}
```

输出为

```
Thread[t1,5,main] : 0
Thread[t1,5,main] : 1
Thread[t1,5,main] : 2
Thread[t1,5,main] : 3
Thread[t1,5,main] : 4
Thread[t3,5,main] : 0 true
Thread[t3,5,main] : 1 false
Thread[t3,5,main] : 2 false
Thread[t3,5,main] : 3 false
Thread[t3,5,main] : 4 false
Thread[t2,5,main] : 0
Thread[t2,5,main] : 1
Thread[t2,5,main] : 2
Thread[t2,5,main] : 3
Thread[t2,5,main] : 4
```

可见t1先于t3执行(不一定，但是肯定不是t3独占式运行)，t2最后执行。

而且可见t3的`Thread.interrupted()`方法第一次检测到标识信号，输出为true，然后擦除掉了，以后输出为false

## 七、线程的关闭

线程在生命周期正常结束时就会关闭，即run方法执行完毕后关闭(ru中的循环不算，因为尚未执行完毕)。

也可以捕获中断信号关闭线程，例如

```
import java.util.concurrent.TimeUnit;

public class TestMain {
    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(new Runnable() {
            @Override
            public void run() {
                while (Thread.interrupted()) {
                    
                }
                System.out.println("I will exit");
            }
        }, "t1");
        t1.start();
        TimeUnit.MICROSECONDS.sleep(1);
        System.out.println("System will be shutdown");
        t1.interrupt();
    }
}
```

以上代码通过检测中断标识而中断自己，也是正常生命周期结束的一种变体。

也可以使用volatile关键字修饰的开关flag来实现，如下

```
import java.util.concurrent.TimeUnit;

public class TestMain {
    
    static class MyThread extends Thread {
        private volatile boolean closed = false;
        @Override
        public void run() {
            while (!closed && !isInterrupted()) {
               
            }
            System.out.println("I will exit");
        }
        public void close() {
            this.closed = true;
            this.interrupt();
        }
    }
    
    public static void main(String[] args) throws InterruptedException {
        MyThread mt = new MyThread();
        mt.start();
        TimeUnit.MICROSECONDS.sleep(1);
        System.out.println("System will be shutdown");
        //do something
        mt.close();
    }
}
```

输出为

```
System will be shutdown
I will exit
```

注意，本文的代码中可能含有没有用途的变量或者方法，忘记删除，需要读者甄别。