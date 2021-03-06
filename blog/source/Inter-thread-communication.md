title: "线程间通信"
date: 2018-12-21 12:03:44 +0800
update: 2018-12-21 12:03:44 +0800
author: me
cover: "-/images/java.jpg"
tags:
    - Java Core
preview: 前文的叫号机示例为保证线程安全引入了synchronized关键字(获得锁保证同一时间只能有一个线程对共享资源操作)，而为了解决synchorized关键字的不可中断不可超时问题，我们引入了显式锁，显式锁通过wait关键字和closed标志为true来实现加锁让欲获得锁的线程进入阻塞状态以等待，通过notify关键字和closed为false来实现释放锁让阻塞状态的线程竞争锁，实际上这就涉及到了进程间的调度，synchronized关键字不仅仅是加锁保证对共享变量访问的互斥性，还可以配合某些策略或者关键实现进程调度。

---

前文的叫号机示例为保证线程安全引入了synchronized关键字(获得锁保证同一时间只能有一个线程对共享资源操作)，而为了解决synchorized关键字的不可中断不可超时问题，我们引入了显式锁，显式锁通过wait关键字和closed标志为true来实现加锁让欲获得锁的线程进入阻塞状态以等待，通过notify关键字和closed为false来实现释放锁让阻塞状态的线程竞争锁，实际上这就涉及到了进程间的调度，synchronized关键字不仅仅是加锁保证对共享变量访问的互斥性，还可以配合某些策略或者关键实现进程调度。

## 一、单线程间通信

### 1.1 同步阻塞

假定有这样一个系统功能，客户端提交Event到服务器，服务器接收客户请求之后开辟线程处理请求，在经过较为复杂的业务计算后将结果返回给客户端。如果采用同步式阻塞消息处理的话，那么处理流程为

![](/images/article/1549881238354-5d8d1e3b-cc90-4be7-8ed1-0cf9f9c80dd9.png)

缺陷如下：

+ 同步Event提交，客户端等待时间过长(提交Event时长+接受Event创建Thread时长+业务处理时长+返回结果时长)会陷入阻塞，导致二次提交Event耗时过长；
+ 由于客户端提交的Event数量不多，导致系统同时受理业务数量有限，也就是系统的整体吞吐量不高；
+ 这种一个线程处理一个Event的方式，会导致频繁的创建开启和销毁，从而增加系统额外开销；
+ 在业务达到峰值的时候，大量的业务处理线程阻塞会导致频繁的CPU上下文切换，从而降低系统性能。

### 1.2 异步非阻塞

而与之相对应的是异步式非阻塞消息处理，具体做法：由于接受Event与处理Event的线程的操作对象都是Event，那么我们可以定义一个类，将对Event的操作(接受与处理操作)都封装起来，该类内部通过synchronized关键字保证线程安全即接受操作与处理操作在同一时间是能有一个对Event队列进行操作。但是我们的Event队列应该有一个上限，以增加系统的稳定性。

我们首先想到如下策略，就是只利用synchronized关键字，如果放Event的线程获得锁之后先判断Event队列时候满了，如果满了就不能再放了，这个时候该线程应该释放锁，因为取Event的线程还在阻塞着呢，应该让取Event的线程从Event队列中取出一部分再让放线程把Event放进去。那么我们就有了一个思路

+ 如果放Event的线程获得锁判断队列已经满了，那么什么都不做，那么就会释放锁，如果没满，那么放进队列里面再释放锁；
+ 如果取Event的线程获得锁判断队列是空的，那么什么也不做，那么也会释放锁，如果非空，那么就取出来进一步处理。代码如下

```
import java.util.LinkedList;

public class EventQueue {
    private final int max;
    static class Event {
        
    }
    private final LinkedList eventQueue = new LinkedList<>();
    private final static int DEFAULT_MAX_EVENT = 10;
    public EventQueue() {
        this(DEFAULT_MAX_EVENT);
    }
    public EventQueue(int max) {
        this.max = max;
    }
    public void produce(Event zhuan) {
        synchronized(eventQueue) {
            if (eventQueue.size() >= max) {
                // 队列满了，什么也不做，就会执行逻辑释放锁(什么逻辑也没有直接释放)
            } else {
                // 队列没满，就会执行逻辑后再释放锁
                System.out.println("The new Event is submitted.");
                eventQueue.addLast(zhuan);
            }
        }
    }
    public Event consume() {
        synchronized(eventQueue) {
            Event event = null;
            if (eventQueue.isEmpty()) {
                // 队列为空，什么也不做，就会执行逻辑释放锁(什么逻辑也没有直接释放)
            } else {
                // 队列没满，就会执行逻辑后再释放锁
                event = eventQueue.removeFirst();
                System.out.println("The Event in queue that name is " + event + " is taked.");
            }
            return event;
        }
    }
}
```

测试文件为

```
import java.util.concurrent.TimeUnit;

public class ZhuanMain {

    public static void main(String[] args) {
        final EventQueue eventQueue = new EventQueue();
        new Thread(new Runnable() {
            @Override
            public void run() {
                while (true) {
                    eventQueue.produce(new EventQueue.Event());
                }
            }
        }, "Producer").start();
        new Thread(new Runnable() {
            @Override
            public void run() {
                while (true) {
                    eventQueue.consume();
                    try {
                        TimeUnit.MICROSECONDS.sleep(10);  
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        }, "Consumer").start();
    }

}
```

部分输出如下

```
The new Event is submitted.
The new Event is submitted.
The new Event is submitted.
The new Event is submitted.
The new Event is submitted.
The new Event is submitted.
The new Event is submitted.
The new Event is submitted.
The new Event is submitted.
The new Event is submitted.
The Event in queue that name is EventQueue$Event@137f5133 is taked.
The new Event is submitted.
The Event in queue that name is EventQueue$Event@1db8bd2c is taked.
The new Event is submitted.
The Event in queue that name is EventQueue$Event@19873a43 is taked.
The new Event is submitted.
The Event in queue that name is EventQueue$Event@5c85b717 is taked.
The new Event is submitted.
The Event in queue that name is EventQueue$Event@351a837d is taked.
The new Event is submitted.
The Event in queue that name is EventQueue$Event@6d93092f is taked.
The new Event is submitted.
```

可见，需求实现了，满了就不再放，空了就不再取。但是这种方式很落后，完全是两个线程在不断轮询，即无时无刻都在检查队列中是否用Event，再进一步操作，很浪费系统资源啊。我们得改造一下，用通知的办法，即队列满了的时候就把自己阻塞，相当于只要队列满了放Event的线程就歇歇，那么由于wait方法阻塞自己不会独占锁，也就意味着取Event的线程可以获得锁去取Event；如果Event没满，那么放Event的线程就把Event放进去并且通知取Event的线程现在有活干了，等我释放锁之后你就可以去拿了。

也很简单，只需要在produce中的if中让当前线程wait进入阻塞状态，那么取Event的线程就会获得锁退出阻塞状态去运行；然后在else分支中增加唤醒其余线程的notify方法，通知其余线程起来干活。

consume也是同理，代码如下

```
import java.util.LinkedList;

public class EventQueue {
    private final int max;
    static class Event {
        
    }
    private final LinkedList eventQueue = new LinkedList<>();
    private final static int DEFAULT_MAX_EVENT = 10;
    public EventQueue() {
        this(DEFAULT_MAX_EVENT);
    }
    public EventQueue(int max) {
        this.max = max;
    }
    public void produce(Event zhuan) {
        synchronized(eventQueue) {
            if (eventQueue.size() >= max) {
                try {
                    System.out.println("The queue is full.");
                    eventQueue.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            System.out.println("The new Event is submitted.");
            eventQueue.addLast(zhuan);
            eventQueue.notify();
        }
    }
    public Event consume() {
        synchronized(eventQueue) {
            if (eventQueue.isEmpty()) {
                try {
                    System.out.println("The queue is empty.");
                    eventQueue.wait();
                } catch (InterruptedException e) {
                   e.printStackTrace();
                }
            }
            Event event = eventQueue.removeFirst();
            System.out.println("The Event in queue that name is " + event + " is taked.");
            this.eventQueue.notify();
            return event;
        }
    }
}
```

主文件不变，结果为

```
The queue is empty.
The new Event is submitted.
The Event in queue that name is EventQueue$Event@6900a9fc is taked.
The new Event is submitted.
The new Event is submitted.
The new Event is submitted.
The new Event is submitted.
The new Event is submitted.
The new Event is submitted.
The new Event is submitted.
The new Event is submitted.
The new Event is submitted.
The new Event is submitted.
The queue is full.
The Event in queue that name is EventQueue$Event@67c74aac is taked.
The new Event is submitted.
The queue is full.
The Event in queue that name is EventQueue$Event@63f8c28c is taked.
The new Event is submitted.
The queue is full.
The Event in queue that name is EventQueue$Event@26fa4097 is taked.
The new Event is submitted.
The queue is full.
The Event in queue that name is EventQueue$Event@4a5942fd is taked.
The new Event is submitted.
The queue is full.
```

可见，两种方法都可以，只是第二种更“智能”一点。

既然wait和notify方法这么优秀，有必要看一下它的签名

wait的签名为

```
public final void wait() throws InterruptedException	//等价于wait(0)，即永不超时进入阻塞状态，调用了该方法的Object只能被notify唤醒
public final void wait(long timeout) throws InterruptedException	//timeout时间过后进入脱离阻塞进入就绪状态
public final void wait(long timeout, int nanos) throws InterruptedException
```

需要注意的是wait方法必须拥有该对象的monitor，也就是该方法只能在同步方法中使用，即被synchronized关键字修饰的方法或者代码块中使用。

再唠叨一下，当前线程执行了该对象的wait方法后，将会放弃对该monitor的所有权并且进入与该对象关联的wait set(线程休息室，等待notify通知，就不用轮询这种低级的办法)中，即调用wait方法后会释放对monitor线程的所有权，其他线程就有机会争抢对该monitor线程的所有权。

notify的相关签名为

```
public final native void notify()	//唤醒单个正在执行该对象wait方法的线程
public final native void notifyAll()	//唤醒wait set中所有的正在执行该对象wait方法的线程
```

## 二、多线程间通信

指的是取Event和放Event分别有多个线程再干，那么线程间就需要通信

如果在上述代码的基础上增加取或者放Event线程的数量，(为了区分线程，给上述代码加上线程名)如下

```
import java.util.LinkedList;

public class EventQueue {
    private final int max;
    static class Event {
        
    }
    private final LinkedList eventQueue = new LinkedList<>();
    private final static int DEFAULT_MAX_EVENT = 10;
    public EventQueue() {
        this(DEFAULT_MAX_EVENT);
    }
    public EventQueue(int max) {
        this.max = max;
    }
    public void produce(Event zhuan) {
        synchronized(eventQueue) {
            if (eventQueue.size() >= max) {
                try {
                    System.out.println(Thread.currentThread().getName()+ " : The queue is full.");
                    eventQueue.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            System.out.println(Thread.currentThread().getName()+ " : The new Event is submitted.");
            eventQueue.addLast(zhuan);
            eventQueue.notify();
        }
    }
    public Event consume() {
        synchronized(eventQueue) {
            if (eventQueue.isEmpty()) {
                try {
                    System.out.println(Thread.currentThread().getName()+ " : The queue is empty.");
                    eventQueue.wait();
                } catch (InterruptedException e) {
                   e.printStackTrace();
                }
            }
            Event event = eventQueue.removeFirst();
            System.out.println(Thread.currentThread().getName()+ " : The Event in queue that name is " + event + " is taked.");
            this.eventQueue.notify();
            return event;
        }
    }
}
```

主文件为

```
import java.util.concurrent.TimeUnit;

public class ZhuanMain {

    public static void main(String[] args) {
        final EventQueue eventQueue = new EventQueue();
        new Thread(new Runnable() {
            @Override
            public void run() {
                while (true) {
                    eventQueue.produce(new EventQueue.Event());
                }
            }
        }, "Producer").start();
        new Thread(new Runnable() {
            @Override
            public void run() {
                while (true) {
                    eventQueue.consume();
                    try {
                        TimeUnit.MICROSECONDS.sleep(10);  
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        }, "Consumer - 1").start();
        new Thread(new Runnable() {
            @Override
            public void run() {
                while (true) {
                    eventQueue.consume();
                    try {
                        TimeUnit.MICROSECONDS.sleep(10);  
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        }, "Consumer - 2").start();
    }

}
```

如果你多运行几次，那么就会发现以下问题

+ LinkedList为空时执行removeFirst方法
+ LinkedList元素为10时执行addLast方法

存在数据不一致的情况，那么解决的办法(实现多线程间通信的办法)是将if改为while，将notify改为notifyAll(一次性唤醒所有的wait set中的阻塞的线程，他们就会争抢monitor锁，这正是并发的优势)。

最终的EventQueue代码为：

```
import java.util.LinkedList;

public class EventQueue {
    private final int max;
    static class Event {
        
    }
    private final LinkedList eventQueue = new LinkedList<>();
    private final static int DEFAULT_MAX_EVENT = 10;
    public EventQueue() {
        this(DEFAULT_MAX_EVENT);
    }
    public EventQueue(int max) {
        this.max = max;
    }
    public void produce(Event zhuan) {
        synchronized(eventQueue) {
            while (eventQueue.size() >= max) {
                try {
                    System.out.println(Thread.currentThread().getName()+ " : The queue is full.");
                    eventQueue.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            System.out.println(Thread.currentThread().getName()+ " : The new Event is submitted.");
            eventQueue.addLast(zhuan);
            eventQueue.notifyAll();
        }
    }
    public Event consume() {
        synchronized(eventQueue) {
            while (eventQueue.isEmpty()) {
                try {
                    System.out.println(Thread.currentThread().getName()+ " : The queue is empty.");
                    eventQueue.wait();
                } catch (InterruptedException e) {
                   e.printStackTrace();
                }
            }
            Event event = eventQueue.removeFirst();
            System.out.println(Thread.currentThread().getName()+ " : The Event in queue that name is " + event + " is taked.");
            this.eventQueue.notifyAll();
            return event;
        }
    }
}
```

综上，线程间通信，通过synchronized关键字(monitor锁)、wait、notify、notifyAll来实现，synchronized关键字本身实现的是一种轮询式的调度，耗费系统资源，如果它配合其余三个方法，那么真正就会利用通知机制，只会在有必要的时候运行。