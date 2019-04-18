title: "捕获线程运行时异常信息与注入钩子线程"
date: 2018-12-21 16:16:38 +0800
update: 2018-12-21 16:16:38 +0800
author: me
cover: "-/images/java.jpg"
tags:
    - Java Core
preview: 线程在运行时是不允许抛出checked异常的，而且线程运行在自己的上下文中，派生它的线程无法直接获得它运行时出现的异常信息。但是，Java提供了UncaughtExceptionHandler接口，当线程在运行过程中出现异常时，会回掉UncaughtExceptionHandler接口，从而就可以知道哪个线程在运行时出错了以及出现了什么样的错误。

---

## 一、捕获线程运行时异常信息

线程在运行时是不允许抛出checked异常的，而且线程运行在自己的上下文中，派生它的线程无法直接获得它运行时出现的异常信息。但是，Java提供了UncaughtExceptionHandler接口，当线程在运行过程中出现异常时，会回掉UncaughtExceptionHandler接口，从而就可以知道哪个线程在运行时出错了以及出现了什么样的错误。

在Thread类当中，关于处理运行时异常的API共有四个，如下

```
public void setUncaughtExceptionHandler(UncaughtExceptionHandler eh)：为某个特定线程指定UncaughtExceptionHandler
public static void setDefaultUncaughtExceptionHandler(UncaughtExceptionHandler eh)：设置全局的UncaughtExceptionHandler
public UncaughtExceptionHandler getUncaughtExceptionHandler()：获取特定线程的UncaughtExceptionHandler
public static UncaughtExceptionHandler getDefaultUncaughtExceptionHandler()：获取全局的UncaughtExceptionHandler
```

示例如下

```
import java.util.concurrent.TimeUnit;

public class CaptrureThreadException {

    public static void main(String[] args) {
        Thread.setDefaultUncaughtExceptionHandler((t, e) -> {
            System.out.println(t.getName() + " occur exception");
            e.printStackTrace();
        });
        final Thread thread = new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    TimeUnit.SECONDS.sleep(2);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                // 这里会出现unchecked异常
                System.out.println(1 / 0);
            }
        }, "Test-Thread");
        thread.start();
    }

}
```

我们通过`Thread.setDefaultUncaughtExceptionHandler`为线程设置全局的异常捕获回调。该示例输出为

```
Test-Thread occur exception
java.lang.ArithmeticException: / by zero
	at CaptrureThreadException$1.run(CaptrureThreadException.java:19)
	at java.lang.Thread.run(Unknown Source)
```

可以看出，明显捕获出现异常的线程和原因。

## 二、注入钩子线程

JVM进程的退出，是由于JVM进程中没有活跃的非守护线程或者收到了系统的中断信号。向JVM程序注入一个Hook线程，那么在JVM进程退出的时候，Hook线程就会启动执行，通过Runtime可以为JVM注入多个Hook线程，示例代码如下

```
import java.util.concurrent.TimeUnit;

public class ThreadHook {

    public static void main(String[] args) {
        Runtime.getRuntime().addShutdownHook(new Thread() {
            @Override
            public void run() {
                try {
                    System.out.println("The hook thread 1 is running.");
                    TimeUnit.SECONDS.sleep(1);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("The hook thread 1 will exit.");
            }
        });
        Runtime.getRuntime().addShutdownHook(new Thread() {
            @Override
            public void run() {
                try {
                    System.out.println("The hook thread 2 is running.");
                    TimeUnit.SECONDS.sleep(1);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("The hook thread 2 will exit.");
            }
        });
        System.out.println("The main thread will is stopping.");
    }

}
```

输出为

```
The main thread will is stopping.
The hook thread 1 is running.
The hook thread 2 is running.
The hook thread 2 will exit.
The hook thread 1 will exit.
```

可见，在主线程退出后Hook线程就会运行，待Hook线程运行结束后，整个JVM进程退出。

Hook的实际意义在哪？比如为了防止某个程序被重复启动，在进程启动时会创建一个lock文件，在又一次启动程序时，程序会检查是否存在lock文件，如果存在，那么说明程序已经启动不能重复启动，如果不存在那么就启动。但是为了下一次正常启动程序，我们得在程序关闭的时候删除掉lock文件，那么Hook线程就派上用场了。

事实上，在MySQL服务器、zookeeper、kafka等系统中都能看到lock文件的存在。