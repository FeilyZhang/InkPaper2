title: "ThreadGroup详解"
date: 2018-12-21 15:40:03 +0800
update: 2018-12-21 15:40:03 +0800
author: me
cover: "-/images/java.jpg"
tags:
    - Java Core
preview: 每个线程都会有ThreadGroup，如果在启动的时候没有在构造方法Thread显式的指明ThreadGroup，那么该线程就会自动的被加入与父线程相同的ThreadGroup当中，启动该线程的线程就是该线程的父线程。

---

每个线程都会有ThreadGroup，如果在启动的时候没有在构造方法Thread显式的指明ThreadGroup，那么该线程就会自动的被加入与父线程相同的ThreadGroup当中，启动该线程的线程就是该线程的父线程。

不仅仅Thread存在父子关系，ThreadGroup也存在父子关系，ThreadGroup中可以有子线程，也可以有子ThreadGroup。

创建ThreadGroup的语法如下

```
public ThreadGroup(String name);	//name指定ThreadGroup的名字，但是该ThreadGroup的父ThreadGroup是创建它的线程所在的ThreadGroup
public ThreadGroup(ThreadGroup parent, String name)	//parent指明了该ThreadGroup的父ThreadGroup，name指明名字
```

示例代码如下

```
public class ThreadGroupTest {

    public static void main(String[] args) {
        // 获取当前线程(主线程)所在的ThreadGroup
        ThreadGroup currentGroup = Thread.currentThread().getThreadGroup();
        // 没有显式的指明该ThreadGroup的父ThreadGroup，所以其父ThreadGroup为创建它的线程(主线程)的ThreadGroup，即currentGroup
        ThreadGroup group1 = new ThreadGroup("Group1");
        System.out.println(currentGroup == group1.getParent()); //true
        // 指明该ThreadGroup的父ThreadGroup为group1
        ThreadGroup group2 = new ThreadGroup(group1, "Group2");
        System.out.println(group2.getParent() == group1);   //true
        System.out.println(group2.getParent().getParent() == group1.getParent());   //true
        System.out.println(group2.getParent() == group1.getParent());   //false
    }

}
```

都很容易理解，不解释

那么该示例的ThreadGroup结构如下

![](/images/article/1549881446369-1daf81e3-77e2-4dbc-a839-63d2016f55f1.png)

<a name="59d3d021"></a>
## 一、Thread数组

顾名思义，Thread数组中的元素是Thread，我们可以通过ThreadGroup的以下方法将该ThreadGroup中的active状态的线程全部复制到Thread数组中

```
public int enumerate(Thread[] list)
public int enumerate(Thread[] list, boolean recurse)
```

如果recurse为true，那么会递归当前ThreadGroup的子ThreadGroup中的active状态的线程到list中。

演示一下

```
import java.util.concurrent.TimeUnit;

public class ThreadGroupEnumerateThreads {

    public static void main(String[] args) throws InterruptedException {
        ThreadGroup myGroup = new ThreadGroup("MyGroup");
        Thread thread = new Thread(myGroup, new Runnable() {

            @Override
            public void run() {
                while (true) {
                    try {
                        TimeUnit.SECONDS.sleep(1);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
            
        }, "MyThread");
        thread.start();
        TimeUnit.MICROSECONDS.sleep(2);
        ThreadGroup mainGroup = Thread.currentThread().getThreadGroup();
        Thread[] list = new Thread[mainGroup.activeCount()];
        int recurseSize = mainGroup.enumerate(list, false);
        System.out.println(recurseSize);    //1
        recurseSize = mainGroup.enumerate(list, true);
        System.out.println(recurseSize);    //2
    }

}
```

第一个输出为1，这是因为mainGroup有一个main线程，一个myGroup，由于boolean为false，那么没有递归myGroup中的Thread，所以为1。那么第二个为2就不难理解了。

## 二、ThreadGroup数组

顾名思义，ThreadGroup数组存放的元素为ThreadGroup，可以用ThreadGroup的以下方法将当前ThreadGroup的子ThreadGroup复制进去。

```
public int enumerate(ThreadGroup[] list)
public int enumerate(ThreadGroup[] list, boolean recurse)
```

仍然是recurse为true递归复制当前ThreadGroup的子ThreadGroup中的ThreadGroup。示例如下

```
import java.util.concurrent.TimeUnit;

public class ThreadGroupEnumerateThreadGroups {

    public static void main(String[] args) throws InterruptedException {
        ThreadGroup group1 = new ThreadGroup("Group1");
        ThreadGroup group2 = new ThreadGroup(group1, "Group2");
        TimeUnit.MILLISECONDS.sleep(2);
        ThreadGroup mainGroup = Thread.currentThread().getThreadGroup();
        ThreadGroup[] list = new ThreadGroup[mainGroup.activeGroupCount()];
        int recurseSize = mainGroup.enumerate(list, false);
        System.out.println(recurseSize);    //1
        recurseSize = mainGroup.enumerate(list, true);
        System.out.println(recurseSize);    //2
    }

}
```

都很简单，不用解释。

## 三、ThreadGroup操作

ThreadGroup并不能提供对线程的管理，ThreadGroup的主要功能是对线程进行组织，其主要方法如下

+ `activeCount()`用于获取group中活跃的线程数量，只是一个估计值；
+ `activeGroupCount()`用于获取group中活跃的子group的线程数量，也是一个估计值；
+ `getMaxPriority()`用于获取group的优先级，不会大于父group的最大优先级；
+ `getName()`用于获取该group的名字；
+ `getParent()`用于获取该group的父ThreadGroup，如果不存在则返回null；
+ `list()`没有返回值，直接将group中活跃的线程信息全部输出到控制台；
+ `parentOf(ThreadGroup g)`会判断当前group是不是给定group的父group，如果给定的group是本身，那么也会返回true；
+ `setMaxPriority(int pri)`会指定group的最大优先级，最大优先级不能超过父ThreadGroup的最大优先级，执行该方法不仅会改变当前group的最大优先级，还会改变当前group的子group的最大优先级；
+ `interrupt()`方法会导致该group中所有的active线程被interrupt，也就是该group中所有的线程的interrupt标识都会被设置为true；
+ `destroy()`方法用于销毁ThreadGroup，该方法只是针对一个没有任何active线程的group进行一次destroy标记，调用该方法的直接结果就是在父group中将自己移除；
+ `isDestroy()`方法判断当前ThreadGroup线程是否被destroy了；
+ `setDaemon()`方法将当前ThreadGroup设置为守护ThreadGroup，那么在group中没有任何active线程的时候该group将自动destroy。