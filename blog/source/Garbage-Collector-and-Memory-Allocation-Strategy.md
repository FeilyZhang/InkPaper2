title: "垃圾收集器与内存分配策略"
date: 2019-04-25 10:31:14 +0800
update: 2019-04-25 10:31:14 +0800
author: me
cover: "-/images/jvm-gc.PNG"
tags:
    - Java Core
preview: 堆中几乎存放着Java中的所有对象实例，垃圾收集器在对堆进行回收前，要做的一件事情就是判断这些对象有哪些还“或者”，哪些已经“死去（即不可能再被任何途径使用的对象）”。然后对死去的对象进行垃圾清理完成内存回收。

---

堆中几乎存放着Java中的所有对象实例，垃圾收集器在对堆进行回收前，要做的一件事情就是判断这些对象有哪些还“或者”，哪些已经“死去（即不可能再被任何途径使用的对象）”。然后对死去的对象进行垃圾清理完成内存回收。

## 一、对象是死是活？

### 1.1 引用计数法

引用计数法（Reference Counting）是判断对象是否存活的一个办法，具体做法是：给对象中添加一个引用计数器，每当有一个地方引用它时，计数器就加1；当引用失效时，计数器值就减一；任何时候计数器都为0的对象就是不可能再被使用的。

这种办法的优点是实现简单，但是缺点是难以解决对象之间的相互循环引用问题。例如

```
objA.instance = objB;
objB.instance = objA'
```

对象objA与objB相互引用对方，使得二者的引用计数器不为0.于是无法通知GC收集器回收它们。

### 1.2 根搜索算法

在主流的商用程序语言中，都是使用根搜索算法（GC Roots Tracing）判断对象是否存活的。这个算法的基本思路是通过一系列名为“GC Roots”的对象作为起始点，从这些节点开始向下搜索，搜索所走过的路径称为引用链（Reference Chain），当一个对象到GC Roots没有任何引用链时或不可达时，则证明该对象是不可用的，它们会被判定为可回收的对象。这里的对象指的是

+ 虚拟机栈（栈帧中的本地变量表）中引用的对象，即堆区中的对象；
+ 方法区中类静态属性引用的对象；
+ 方法区中的常量引用的对象；
+ 本地方法栈JNI（即一般所说的Native方法）的引用的对象。

![](/images/article/gc-roots.png)

### 1.3 再谈引用

无论是通过引用计数算法判断对象的引用数量，还是通过根搜索算法判断对象的引用链是否可达，可见，判定对象是否存活都与“引用有关”。在JDK1.2之前，Java中的引用的定义很传统：如果reference类型的护具中存储的数值代表的是另外一块内存的起始地址，就称这块内存代表着一个引用。但是这种定义很简单纯粹，只有被引用雨没有被引用两种状态，对于一些“食之无味，弃之可惜”的对象就显得无能为力。我们希望能描述这样一类对象：当内存空间还足够时，则能保存在内存之中；如果内存在进行垃圾收集后还是非常紧张，则可以抛弃这些对象。

在JDK1.2之后，Java对引用的概念进行了扩充，将引用分为强引用（Strong Reference）、软引用（Soft Reference）、弱引用（Weak Reference）、虚引用（Phantom Reference）四种，这四种引用强度依次逐渐减弱。

+ 强引用就是指程序代码之中普遍存在的，类似于`Object obj = new Object()`之类的引用，只要强引用还存在，那么垃圾收集器永远都不会回收；
+ 软引用是用来描述一些还有用，但并非必需的对象。对于软引用的对象，在系统将要发生内存溢出异常之前，将会把这些对象列入垃圾回收范围之内并进行第二次回收。如果这次回收还是没有足够的内存，才会抛出内存溢出异常。在JDK1.2之后，提供了`SoftReference`类来实现软引用.
+ 弱引用也是用来描述非必需对象的，但是它的强度比软引用更弱一些，被弱引用关联的对象只能生存到下一次垃圾收集发生之前。当垃圾收集器工作时，无论当前内存是否足够，都会回收掉只被弱引用关联的对象（这也是区别于软引用的地方）。在JDK1.2之后，提供了`WeakReference`类来实现弱引用；
+ 虚引用也成为幽灵引用或者幻影引用，是最弱的的一种引用关系。一个对象是否有虚引用的存在，完全不会对其生存时间构成影响，也无法通过虚引用来取得一个对象实例。为一个对象设置虚引用关联的唯一目的就是希望能在这个对象被收集器回收时收到一个系统通知。在JDK1.2之后，提供了`PhantomReference`类来实现虚引用。

### 1.4 生存还是死亡

在根搜素算法中，不可达的对象并非非死不可，这时候它们只是进入了缓刑阶段，要真正宣告一个对象的死亡，至少需要两次标记过程：如果对象在进行根搜索后发现没有与GC Roots相连接的引用链，那么它将会被标记并且进行一次筛选，筛选的条件是此对象是否有必要执行`finalize()`方法，当对象没有覆盖这个方法时或者该方法已经被虚拟机调用过，虚拟机将这两种情况都视为“没有必要执行”。

如果这个对象被判定为有必要执行`finalize()`方法，那么这个对象将会被放置在一个名为`F-Queue`的队列之中，并在稍后由一条由虚拟机自动建立的、低优先级的`Finalizer`线程去执行。这里所谓的执行是指虚拟机会触发这个方法但并不承诺会等待它运行结束。这样做的原因是，如果一个对象在`Finalize()`方法中执行缓慢或者发生了死循环，将很可能会导致`F-Queue`队列中的其他对象永久处于等待状态，甚至导致整个内存回收系统崩溃。

`finalize()`方法是对象逃脱死亡命运的最后一次机会，稍后GC将对`F-Queue`中的对象进行第二次小规模标记，如果对象要在`finalize()`中成功拯救自己，只需要重新与引用链上的任何一个对象建立关联即可。譬如，把自己(this关键字)赋给某个类变量或对象的成员变量（随便一个类都可以，包括自己），那么在第二次标记时它将被移除出“即将回收”的集合；如果对象这时候还没有逃脱，那么就真的离死不远了。示例如下

```
import java.util.concurrent.TimeUnit;

public class FinalizeEscapeGC {

    public static FinalizeEscapeGC SAVE_HOOK = null;
    
    public void isLive() {
        System.out.println("yes, i am still live.");
    }
    
    @Override
    protected void finalize() throws Throwable {
        super.finalize();
        System.out.println("finalize method executed.");
        // 把自己赋给FinalizeEscapeGC型对象SAVE_HOOK,从而拯救自己
        FinalizeEscapeGC.SAVE_HOOK = this;
    }
    public static void main(String[] args) throws InterruptedException {
        SAVE_HOOK = new FinalizeEscapeGC();
        
        // 对象第一次成功拯救自己
        SAVE_HOOK = null;
        System.gc();
        // 因为Finalizer方法优先级很低，主线程暂停0.5s，以等待它
        TimeUnit.SECONDS.sleep(5);
        if (SAVE_HOOK != null) {
            SAVE_HOOK.isLive();
        } else {
            System.out.println("no, i am dead.");
        }
        
        // 下面这段代码与上面的完全相同，但是却拯救失败了
        SAVE_HOOK = null;
        System.gc();
        // 因为Finalizer方法优先级很低，主线程暂停0.5s，以等待它
        TimeUnit.SECONDS.sleep(5);
        if (SAVE_HOOK != null) {
            SAVE_HOOK.isLive();
        } else {
            System.out.println("no, i am dead.");
        }
    }

}
```

结果为

```
finalize method executed.
yes, i am still live.
no, i am dead.
```

可以看到，SAVE_HOOK对象的`finalize()`方法确实被GC收集器触发过，并且在被收集之前成功逃脱了。

第二次执行失败是因为`finalize()`方法只会被系统自动执行一次，第二次无效，所以自救失败了。