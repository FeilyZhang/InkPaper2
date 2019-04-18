title: "线程安全与数据同步"
date: 2018-12-20 16:53:18 +0800
update: 2018-12-20 16:53:18 +0800
author: me
cover: "-/images/java.jpg"
tags:
    - Java Core
preview: 如果用synchronized关键字修饰一个方法或者代码块，那么就可以防止因多线程作用共享资源而导致的数据不一致或者线程安全问题，该关键字只会保证其作用的方法或者代码块在同一时间只会被一个线程使用，即被修饰对象只对获得锁的线程可见，其余线程只能阻塞等待锁的释放。

---

## 一、synchronized关键字初识

在《线程间共享变量与synchronized关键字解决线程不安全问题》一文当中的叫号机程序由于多线程对共享变量(共享资源)的访问导致数据不一致问题，我们是通过synchronized关键字来解决的。

如果用synchronized关键字修饰一个方法或者代码块，那么就可以防止因多线程作用共享资源而导致的数据不一致或者线程安全问题，该关键字只会保证其作用的方法或者代码块在同一时间只会被一个线程使用，即被修饰对象只对获得锁的线程可见，其余线程只能阻塞等待锁的释放。其具体表现如下

+ synchronized关键字提供了一种锁机制，能够保证多线程对共享变量的互斥访问，从而避免了数据不一致(线程不安全)的情况；
+ synchronized关键字包括monitor enter和monitor exit两个JVM指令，成对出现，他们能够保证在任何时候任何线程执行到monitor enter成功之前即获得锁之时都必须从主内存中获取数据，而不是从缓存中获取数据，在monitor exit运行成功之后即释放锁之后，共享变量被更新后的值必须刷u入主内存之中;
+ synchronized关键字的指令严格按照java happens-before 规则，一个monitor exit指令之前必须要有一个monitor enter。

synchronized关键字的用法如下

一种是修饰在方法上，此时方法就会变成同步方法，即多个线程只能一个一个进入，且一个进入另一个只能等待，如下

```
[default|public|private|protected] synchronized [static] type method() {
    
}
```

另一种是修饰在同步代码块上，如下

```
private final Object MUTEX = new Object();
public void sync() {
    synchronized(MUTEX) {
        //设计共享变量的相关逻辑
    }
}
```

## 二、synchronized关键字的注意问题

**第一个注意的是，与monitor关联的对象不能为空，**如下

```
private final Object MUTEX= null;
public void sync() {
    synchronized(MUTEX) {
        //
    }
}
```

以上是不对的，应该实例化MUTEX，否则就等于没加锁

**第二个应该注意的是，synchronized的粒度不能过大，否则就会失去并发的优势。**

因为多线程在多核处理器下本来是并行运行的，所以速度快，但是到涉及到共享变量的被synchronized修饰的同步代码块时，就会变成串行执行，如果粒度过大，那么串行执行的代码越多，就会丧失并发的优势。

所以，尽可能锁涉及共享变量操作的代码块，而尽可能不锁方法。

**第三个应该注意的是，不同的monitor企图锁相同的方法，这也是不对的**

简单的说，锁存在的意义就是多线程之间必须涉及到对共享变量的操作，如果每个线程都构造一个有着相同代码(或者相同方法)的Runnable实例，那么实例只会存在于线程的私有作用域，压根不会涉及到对共享变量的操作，举个例子，就用之前的叫号机，如下

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

public class TicketWindowMain {

    public static void main(String[] args) {
        //TicketWindow tw = new TicketWindow();
        Thread t1 = new Thread(new TicketWindow(), "First");
        Thread t2 = new Thread(new TicketWindow(), "Second");
        Thread t3 = new Thread(new TicketWindow(), "Third");
        t1.start();
        t2.start();
        t3.start();
        
    }

}
```

输出为

```
Thread[First,5,main] current number is 1
Thread[First,5,main] current number is 2
Thread[Third,5,main] current number is 1
Thread[Third,5,main] current number is 2
Thread[Third,5,main] current number is 3
Thread[Third,5,main] current number is 4
Thread[Third,5,main] current number is 5
Thread[Third,5,main] current number is 6
Thread[Third,5,main] current number is 7
Thread[Third,5,main] current number is 8
Thread[Second,5,main] current number is 1
Thread[Second,5,main] current number is 2
Thread[Second,5,main] current number is 3
Thread[Second,5,main] current number is 4
Thread[Second,5,main] current number is 5
Thread[Second,5,main] current number is 6
Thread[Second,5,main] current number is 7
Thread[Second,5,main] current number is 8
Thread[Second,5,main] current number is 9
Thread[Second,5,main] current number is 10
Thread[Third,5,main] current number is 9
Thread[Third,5,main] current number is 10
Thread[First,5,main] current number is 3
Thread[First,5,main] current number is 4
Thread[First,5,main] current number is 5
Thread[First,5,main] current number is 6
Thread[First,5,main] current number is 7
Thread[First,5,main] current number is 8
Thread[First,5,main] current number is 9
Thread[First,5,main] current number is 10
```

可见，由于每个线程都有了变量num，使得num不再是共享变量，就会导致加锁失败。解决问题的方法就是让诸线程使用同一个对象实例，即让多线程使用同一个包含共享变量方法的类实例，如下改动主文件

```
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
Thread[Second,5,main] current number is 1
Thread[Second,5,main] current number is 2
Thread[Second,5,main] current number is 3
Thread[Second,5,main] current number is 4
Thread[Second,5,main] current number is 5
Thread[First,5,main] current number is 6
Thread[First,5,main] current number is 7
Thread[First,5,main] current number is 8
Thread[First,5,main] current number is 9
Thread[First,5,main] current number is 10
```

**第四个应该注意的是，多个锁交叉导致的死锁**

如果A线程持有R1的锁而等待R2的锁，B线程正好持有R2的锁而等待R1的锁，那么就会形成死锁。示例如下

```
public class DeadLock {
    private final Object READ_MUTEX = new Object();
    private final Object WRITE_MUTEX = new Object();
    
    public void read() {
        synchronized(READ_MUTEX) {
            System.out.println(Thread.currentThread().getName() + "get READ lock");
            synchronized(WRITE_MUTEX) {
                System.out.println(Thread.currentThread().getName() + "get WRITE lock");
            }
            System.out.println(Thread.currentThread().getName() + "release WRITE lock");
        }
        System.out.println(Thread.currentThread().getName() + "release READ lock");
    }
    
    public void write() {
        synchronized(WRITE_MUTEX) {
            System.out.println(Thread.currentThread().getName() + "get WRITE lock");
            synchronized(READ_MUTEX) {
                System.out.println(Thread.currentThread().getName() + "get READ lock");
            }
            System.out.println(Thread.currentThread().getName() + "release READ lock");
        }
        System.out.println(Thread.currentThread().getName() + "release WRITE lock");
    }
    
    public static void main(String[] args) {
        final DeadLock dl = new DeadLock();
        new Thread(new Runnable() {
            @Override
            public void run() {
                while (true) {
                    dl.read();
                }
            }
            
        }, "READ_Thread ").start();
        new Thread(new Runnable() {
            @Override
            public void run() {
                while (true) {
                    dl.write();
                }
            }
            
        }, "WRITE_Thread ").start();
    }
}
```

多运行几次，就会有如下输出

```
READ_Thread get READ lock
WRITE_Thread get WRITE lock
```

可见READ_Thread已经获得READ_LOCK等待获得WRITE_LOCK,而WRITE_Thread已经获得WRITE_LOCK,等待获得READ_LOCK，双方同时拥有对方所需要的资源导致死锁，所以一定要注意不能多个锁交叉。

另外，使用锁的前提是存在共享资源，那么势必多个线程只用一个RUNNABLE实例，如果把上述代码改动一下，如下

```
public class DeadLock {
    private final Object READ_MUTEX = new Object();
    private final Object WRITE_MUTEX = new Object();
    
    public void read() {
        synchronized(READ_MUTEX) {
            System.out.println(Thread.currentThread().getName() + "get READ lock");
            synchronized(WRITE_MUTEX) {
                System.out.println(Thread.currentThread().getName() + "get WRITE lock");
            }
            System.out.println(Thread.currentThread().getName() + "release WRITE lock");
        }
        System.out.println(Thread.currentThread().getName() + "release READ lock");
    }
    
    public void write() {
        synchronized(WRITE_MUTEX) {
            System.out.println(Thread.currentThread().getName() + "get WRITE lock");
            synchronized(READ_MUTEX) {
                System.out.println(Thread.currentThread().getName() + "get READ lock");
            }
            System.out.println(Thread.currentThread().getName() + "release READ lock");
        }
        System.out.println(Thread.currentThread().getName() + "release WRITE lock");
    }
    
    public static void main(String[] args) {
        final DeadLock dl = new DeadLock();
        new Thread(new Runnable() {
            @Override
            public void run() {
                while (true) {
                    new DeadLock().read();
                }
            }
            
        }, "READ_Thread ").start();
        new Thread(new Runnable() {
            @Override
            public void run() {
                while (true) {
                    new DeadLock().write();
                }
            }
            
        }, "WRITE_Thread ").start();
    }
}
```

那么就会产生不同的monitor企图锁相同的方法的问题，一定要注意

## 三、自定义显式锁BooleanLock

synchronized关键字存在缺陷，第一个缺陷是无法控制阻塞时长，即锁被另一个线程占用，那么当前线程试图获取锁那么就会进入阻塞状态，如果占用锁的线程占用时间很长，那么也就意味着当前线程阻塞的时间就很长，很显然，synchronized关键字无法控制阻塞时长，另一个缺点是阻塞不可被中断，即如果当前线程因争夺monitor锁而进入阻塞主公你太，那么进入阻塞状态的线程是无法中断的，虽然可以使用interrupt进行中断，但是synchronized关键字不像sleep和wait方法一样可以捕获中断信号，所以这个缺点问题也是很大的。

先演示第一个问题

```
import java.util.concurrent.TimeUnit;

public class SynchronizedDefect {
    public synchronized void syncMethod() {
        try {
            TimeUnit.HOURS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
    public static void main(String[] args) throws InterruptedException {
        SynchronizedDefect sd = new SynchronizedDefect();
        Thread t1 = new Thread(new Runnable() {
            @Override
            public void run() {
                sd.syncMethod();
            }
        });
        t1.start();
        TimeUnit.MICROSECONDS.sleep(2);
        Thread t2 = new Thread(new Runnable() {
            @Override
            public void run() {
                sd.syncMethod();
            }
        });
        t2.start();
    }
}
```

一定要注意，多线程对锁的操作涉及到共享变量，一定要使用同一个Runnable实例。

我们在被锁住的方法中让当前线程睡眠1h，先启动t1线程，为了保证t1线程最进入同步方法，我们让主线程睡眠2ms再启动t2线程，那么由于t1线程占用锁休眠期间并不会释放锁，那么t2线程就要无限等待1h等t1释放锁之后才能获得锁，这样也就是说synchronized关键字无法控制阻塞时长，即超过阻塞时间就放弃锁进入就绪状态而不是等待锁在阻塞状态。

再演示第二个问题

```
import java.util.concurrent.TimeUnit;

public class SynchronizedDefect {
    public synchronized void syncMethod() {
        try {
            TimeUnit.HOURS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
    public static void main(String[] args) throws InterruptedException {
        SynchronizedDefect sd = new SynchronizedDefect();
        Thread t1 = new Thread(new Runnable() {
            @Override
            public void run() {
                sd.syncMethod();
            }
        });
        t1.start();
        TimeUnit.MICROSECONDS.sleep(2);
        Thread t2 = new Thread(new Runnable() {
            @Override
            public void run() {
                sd.syncMethod();
            }
        });
        t2.start();
        t2.interrupt();
        System.out.println(t2.isInterrupted());	//true
        System.out.println(t2.getState());	//BLOCKED
    }
}
```

t2进入阻塞状态，那么主线程运行，主线程试图通过t2的interrupt方法打断t2的阻塞状态使其进入就绪状态，但是很稀罕，从输出来看interrupt确实中断过t2线程，但是失败了，其状态仍然为BLOCKED，这就说明了Synchronized关键字无法被中断。

面对Synchronized的这两个缺陷，我们可以通过java.util.concurrent包中的显式锁ReentrantLock来实现，主要方法如下

```
ReentrantLock lock = new ReentrantLock();
 
// 获取锁，这是跟synchronized关键字对应的用法。
lock.lock();
try{
  // your code
}finally{
  lock.unlock();
}
 
// 可定时，超过指定时间没得到锁就放弃
try {
  lock.tryLock(10, TimeUnit.SECONDS);
  try {
    // your code
  }finally {
    lock.unlock();
  }
} catch (InterruptedException e1) {
  // exception handling
}
 
// 可中断，等待获取锁的过程中线程线程可被中断
try {
  lock.lockInterruptibly();
  try {
    // your code
  }finally {
    lock.unlock();
  }
} catch (InterruptedException e) {
  // exception handling
}
```

另外，也可以自定义显式锁BooleanLock来实现，如下

我们先定义锁的接口(当然，也可以不定义直接写，规范嘛，易拓展和理清思路)

```
import java.util.List;
import java.util.concurrent.TimeoutException;

public interface Lock {
    public abstract void lock() throws InterruptedException;
    public abstract void lock(long mills) throws InterruptedException, TimeoutException;
    public abstract void unlock();
    List getBlockThreads();
}
```

实现了该接口的代码为

```
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;
import java.util.concurrent.TimeoutException;
import static java.lang.Thread.currentThread;

public class BooleanLock implements Lock {
    // 当前拥有锁的线程，一般是调用过lock等方法后获得锁的线程
    private Thread currentHasThread;
    // 当前锁是否被获取
    private boolean locked = false;
    // 阻塞的线程列表
    private final List blockedThreads = new ArrayList<>();
    @Override
    public void lock() throws InterruptedException {
        synchronized (this) {
            // 如果当前锁已经被获取且阻塞列表中没有自己，那么将当前线程加入到阻塞列表中，并调用wait方法让当前线程进入阻塞状态等待锁的释放
            while (locked) {
                if (!blockedThreads.contains(currentThread())) {
                    blockedThreads.add(currentThread());
                }
                this.wait();
            }
            // 该线程获得锁之后while判断无效，继续从这里执行
            // 由于当前线程已经获得锁，那么将自身从阻塞列表中移除
            // 并将标记locked设置为true代表自己已经拿到了锁，其余线程想要拿锁只能阻塞等待，然后将当前拥有锁的线程设置为自己
            blockedThreads.remove(currentThread());
            this.locked = true;
            this.currentHasThread = currentThread();
        }
    }

    @Override
    public void lock(long mills) throws InterruptedException, TimeoutException {
        synchronized (this) {
            // 如果mills小于0，那么自动调用lock方法处理
            if (mills <= 0) {
                this.lock();
            } else {
                //  经过一系列时间判断之后再决定是否超时放弃获得锁
                long remainingMills = mills;
                long endMills = System.currentTimeMillis() + remainingMills;
                while (locked) {
                    if (remainingMills <= 0) {
                        throw new TimeoutException("can not get the lock during " + mills + " ms.");
                    }
                    if (!blockedThreads.contains(currentThread())) {
                        blockedThreads.add(currentThread());
                    }
                    this.wait(remainingMills);
                    remainingMills = endMills - System.currentTimeMillis();
                }
                blockedThreads.remove(currentThread());
                this.locked = true;
                this.currentHasThread = currentThread();
            }
        }
    }

    @Override
    public void unlock() {
        synchronized (this) {
            // 如果是自己拿着锁，那么直接将状态设置为true，并通知所有的阻塞线程抢占锁
            if (currentHasThread == currentThread()) {
                this.locked = false;
                this.notifyAll();
            }
        }
    }

    @Override
    public List getBlockThreads() {
        return Collections.unmodifiableList(blockedThreads);
    }

}
```

可以测试一下

```
import static java.lang.Thread.currentThread;
import static java.util.concurrent.ThreadLocalRandom.current;

import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.ReentrantLock;
import java.util.stream.IntStream;
public class BooleanLockTest {
    private final ReentrantLock lock = new ReentrantLock();
    public void syncMethod() {
        lock.lock();
        try {
            int randomInt = current().nextInt(5);
            System.out.println(currentThread() + "get the lock.");
            try {
                TimeUnit.SECONDS.sleep(randomInt);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(currentThread() + "will release the lock monitor.");
        } finally {
            lock.unlock();
        }
    }
    public static void main(String[] args) {
        BooleanLockTest bt = new BooleanLockTest();/*
        for (int i = 0; i < 10; i++) {
            new Thread(new Runnable() {
                @Override
                public void run() {
                    bt.syncMethod();
                }
            }, "My-Thread-" + i).start();
        }*/
        IntStream.range(0, 10).mapToObj(i -> new Thread(bt::syncMethod)).forEach(Thread::start);
    }
}
```

这是current包内显式锁的测试，也可以改为我们自定义的，如下

```
import static java.lang.Thread.currentThread;
import static java.util.concurrent.ThreadLocalRandom.current;

import java.util.concurrent.TimeUnit;
import java.util.stream.IntStream;
public class BooleanLockTest {
    private final Lock lock = new BooleanLock();
    public void syncMethod() throws InterruptedException {
        lock.lock();
        try {
            int randomInt = current().nextInt(5);
            System.out.println(currentThread() + "get the lock.");
            try {
                TimeUnit.SECONDS.sleep(randomInt);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(currentThread() + "will release the lock monitor.");
        } finally {
            lock.unlock();
        }
    }
    public static void main(String[] args) {
        BooleanLockTest bt = new BooleanLockTest();/*
        for (int i = 0; i < 10; i++) {
            new Thread(new Runnable() {
                @Override
                public void run() {
                    bt.syncMethod();
                }
            }, "My-Thread-" + i).start();
        }*/
        IntStream.range(0, 10).mapToObj(i -> new Thread(() -> {
            try {
                bt.syncMethod();
            } catch (InterruptedException e) {
                // TODO Auto-generated catch block
                e.printStackTrace();
            }
        })).forEach(Thread::start);
    }
}
```

```
Thread[Thread-0,5,main]get the lock.
Thread[Thread-0,5,main]will release the lock monitor.
Thread[Thread-9,5,main]get the lock.
Thread[Thread-9,5,main]will release the lock monitor.
Thread[Thread-1,5,main]get the lock.
Thread[Thread-1,5,main]will release the lock monitor.
Thread[Thread-8,5,main]get the lock.
Thread[Thread-8,5,main]will release the lock monitor.
Thread[Thread-2,5,main]get the lock.
Thread[Thread-2,5,main]will release the lock monitor.
Thread[Thread-7,5,main]get the lock.
Thread[Thread-7,5,main]will release the lock monitor.
Thread[Thread-3,5,main]get the lock.
Thread[Thread-3,5,main]will release the lock monitor.
Thread[Thread-6,5,main]get the lock.
Thread[Thread-6,5,main]will release the lock monitor.
Thread[Thread-4,5,main]get the lock.
Thread[Thread-4,5,main]will release the lock monitor.
Thread[Thread-5,5,main]get the lock.
Thread[Thread-5,5,main]will release the lock monitor.
```

`lock()`与`unluck()`的组合就相当于`synchronized`关键字的作用。我们可以看到输出结果是一个释放了锁另一个才能获得锁

我们再看一下`lock(long mills)`的用法，如下

```
import static java.lang.Thread.currentThread;
import static java.util.concurrent.ThreadLocalRandom.current;

import java.util.concurrent.TimeUnit;
import java.util.concurrent.TimeoutException;
public class BooleanLockTest {
    private final Lock lock = new BooleanLock();
    public void syncMethod() throws InterruptedException {
        try {
            lock.lock(1000);
            int randomInt = current().nextInt(10);
            System.out.println(currentThread() + "get the lock.");
            try {
                TimeUnit.SECONDS.sleep(randomInt);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(currentThread() + "will release the lock monitor.");
        } catch (TimeoutException e1) {
            // TODO Auto-generated catch block
            e1.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
    public static void main(String[] args) throws InterruptedException {
        BooleanLockTest bt = new BooleanLockTest();
        Thread t1 = new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    bt.syncMethod();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });
        Thread t2 = new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    bt.syncMethod();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });
        t1.start();
        TimeUnit.MICROSECONDS.sleep(2);
        t2.start();
       // TimeUnit.MICROSECONDS.sleep(10);
        t2.interrupt();
    }
}
```

输出如下

```
Thread[Thread-0,5,main]get the lock.
java.lang.InterruptedException
	at java.lang.Object.wait(Native Method)
	at BooleanLock.lock(BooleanLock.java:50)
	at BooleanLockTest.syncMethod(BooleanLockTest.java:10)
	at BooleanLockTest$2.run(BooleanLockTest.java:42)
	at java.lang.Thread.run(Unknown Source)
Thread[Thread-0,5,main]will release the lock monitor.
```

我们在启动t1之后让主线程休眠2ms，这样能保证t1先参与CPU竞争，然后启动t2再休眠10ms，保证t2尽快参与CPU竞争，那么我们从输出可以看到主线程中断了t2的信号可以被t2线程接收到，那么自动抛出异常放弃锁不再阻塞。

以上分别是类似于synchronized的用途和可中断synchronized变体，我们如果去掉主线程对t2的中断指令那么就是可超时线程，如下示例

```
import static java.lang.Thread.currentThread;
import static java.util.concurrent.ThreadLocalRandom.current;

import java.util.concurrent.TimeUnit;
import java.util.concurrent.TimeoutException;
public class BooleanLockTest {
    private final Lock lock = new BooleanLock();
    public void syncMethod() throws InterruptedException {
        try {
            lock.lock(1000);
            int randomInt = current().nextInt(10);
            System.out.println(currentThread() + "get the lock.");
            try {
                TimeUnit.SECONDS.sleep(randomInt);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(currentThread() + "will release the lock monitor.");
        } catch (TimeoutException e1) {
            // TODO Auto-generated catch block
            e1.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
    public static void main(String[] args) throws InterruptedException {
        BooleanLockTest bt = new BooleanLockTest();
        Thread t1 = new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    bt.syncMethod();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });
        Thread t2 = new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    bt.syncMethod();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });
        t1.start();
        TimeUnit.MICROSECONDS.sleep(2);
        t2.start();
    }
}
```

输出为

```
Thread[Thread-0,5,main]get the lock.
java.util.concurrent.TimeoutException: can not get the lock during 1000 ms.
	at BooleanLock.lock(BooleanLock.java:45)
	at BooleanLockTest.syncMethod(BooleanLockTest.java:10)
	at BooleanLockTest$2.run(BooleanLockTest.java:42)
	at java.lang.Thread.run(Unknown Source)
Thread[Thread-0,5,main]will release the lock monitor.
```