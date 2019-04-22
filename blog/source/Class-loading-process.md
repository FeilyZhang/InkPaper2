title: "类的加载过程"
date: 2019-04-18 19:31:25 +0800
update: 2019-04-18 20:50:25 +0800
author: me
cover: "-/images/article/class-load-process.jpg"
tags:
    - Java Core
preview: ClassLoader的主要职责就是负责加载各种class文件到JVM中，ClassLoader是一个抽象的class，给定一个class的二进制文件名，ClassLoader会尝试加载并且在JVM中生成构成这个类的各个数据结构，然后使其分布在JVM对应的内存区域中。

---

ClassLoader的主要职责就是负责加载各种class文件到JVM中，ClassLoader是一个抽象的class，给定一个class的二进制文件名，ClassLoader会尝试加载并且在JVM中生成构成这个类的各个数据结构，然后使其分布在JVM对应的内存区域中。

## 一、类的加载过程

类的加载过程一般分为三大步，分别是加载阶段、连接阶段、初始化阶段，如下图所示

![](/images/article/class-load-process.jpg)

> 图片来源：http://www.cnblogs.com/chenpi/p/5393650.html

+ 加载阶段：主要负责查找并且加载类的二进制数据文件，其实就是class文件；
+ 连接阶段：主要包括如下三个方面；
  - 验证：主要是确保类文件的正确性，比如class的版本、魔数等是否正确；
  - 准备：为类的静态变量分配内存，并且为其初始化默认值；
  - 解析：把类中的符号引用转换为直接引用。
+ 初始化阶段：为类的静态变量赋予正确的初始值。

当一个JVM在我们通过执行Java命令启动之后，其中可能包含的类非常多，并不是所有的类都会被初始化。JVM对类的初始化是一个延迟的机制，即使用的是lazy的方式，当一个类在首次**主动使用**时在会被初始化，在同一个运行时包下，一个Class只会被初始化一次。那么什么是主动使用呢？

## 二、主动使用与被动使用

JVM虚拟机规范规定了，每个类或者接口被Java程序首次主动使用时才会对其进行初始化，JVM规范了以下6种主动使用类的场景：

+ 通过`new`关键字会导致相应类的初始化；
+ 访问类的静态`变量`会导致相应类的初始化；
+ 访问类的静态方法会导致相应类的初始化；
+ 对某个类进行反射操作，会导致相应类的初始化；
+ 初始化子类会导致对应父类的初始化（但是需要注意的是，通过子类使用父类的静态变量只会导致父类的初始化，子类不会被初始化）；
+ 启动类，即`main`函数所在的类会导致该类的初始化。

除了以上6种情况，其余均属于被动使用，不会导致类的加载和初始化。

还有两种情况需要注意，分别是

+ 构造某个类的数组时并不会导致该类的初始化；

```
public static void main(String[] args) {
    Simple[] simple = new Simple[10];
	System.out.println(simple.length);
}
```
即上述代码并不会导致类的初始化，虽然仍然使用了new关键字，但是实际上只是建立了一个Simple型的数组，只是在堆内存中开辟了一段4byte × 10的连续地址空间。

+ 引用类的静态常量不会导致类的初始化。

即被final修饰的静态常量被其它类访问是不会导致该类的初始化的。

## 三、类的详细加载过程详解

### 3.1 类的加载阶段

**类的加载就是将class文件中的二进制数据读取到内存中，然后将该字节流所代表的静态存储结构转换为方法区中运行时的数据结构，并且在堆内存中生成一个该类的java.lang.Class对象，作为访问方法区数据结构的入口。**

类加载的最终产物就是堆内存中的class对象，对同一个ClassLoader来讲，不管某个类被加载了多少次，对应到堆内存中的class对象始终是同一个。虚拟机规范中指出了类的加载是通过一个全限定名(包名+类名)来获取二进制数据流，但是并没有限定必须通过何种方式去获得，**常见的是通过class二进制文件的形式**，但除此之外还有其他形式：

+ 运行时动态生成；
+ 通过网络获取；
+ 通过读取zip文件获得类的二进制字节流；
+ 将类的二进制数据压缩在数据库的BLOB字段类型中；
+ 运行时生成class文件，并且动态加载。

### 3.2 类的连接阶段

可以分为三个步骤，分别是验证、准备和解析。

#### 3.2.1 验证

验证的主要目的就是确保class文件的字节流所包含的内容符合当前JVM的规范要求，并且不会出现危害JVM自身安全的代码。当字节流信息不符合要求时，将会抛出VerifyError这样的异常或者其子异常。验证的内容主要包括：

+ 验证文件格式：包括魔数、版本号、MD5指纹、常量池是否存在不被支持的变量类型(比如int64)、指向常量中的引用是否指到了不存在的常量或者该常量的类型不被支持等；
+ 验证元数据（其实就是对class的字节流进行语义分析的过程）：检查该类是否存在父类、检查该类是否继承了被final修饰的类、检查该类是否为抽象类如果是抽象类那么它是否实现了父类的抽象方法或者接口中的所有方法、检查方法重载的合法性以及 其它语义验证；
+ 字节码验证（主要是验证程序的控制流程，比如循环和分支等）：保证当前线程在程序计数器中的指令不会跳转到不合法的字节码指令中去、保证类型转换是合法的、保证任意时刻虚拟机栈中的操作栈类型与指令代码都能正确地被执行、其它验证；
+ 符号引用验证（主要作用是验证符号引用转化为直接引用时的合法性，主要目的就是为了保证解析动作的顺利执行）：通过符号引用描述的字符串全限定名是否能够顺利的找到相关的类、符号引用中的类(、字段、方法)是否对当前类可见等。

#### 3.2.2 准备

当一个class对象的字节流通过了所有的验证过程之后，就开始为该对象的类变量(即静态变量)分配内存并且赋默认值，类变量的内存会被分配到方法区中，不同于实例变量会被分配到堆内存之中。

#### 3.2.3 解析

解析就是在常量池中寻找类、接口、字段和方法的符号应用，并且将这些符号引用替换为直接引用的过程，解析过程中照样会进行一些交叉验证，比如符号引用的验证。

### 3.3 类的初始化阶段

类的初始化阶段就是整个类加载过程中的最后一个阶段，在该阶段最主要的一件事情就是执行`<clinit>()`方法，该方法是在编译阶段生成的，包含了所有的类变量和静态语句块中的代码，在此阶段所有的类变量都会被赋予正确的值。但是需要注意的是，静态语句块只能对后面的静态变量赋值但是不能对其访问，比如

```
public class ClassInit {
    static {
	    System.out.println(x);
		x = 100;
	}
	private static int x = 10;
}
```
以上代码无法编译通过，因为静态语句块只能执行静态变量的赋值但是不能访问静态变量。

另外`<clinit>()`方法与类的构造函数不同，它不需要显示的调用父类的构造器，虚拟机会保证父类的`<clinit>()`方法最先执行，因此父类的静态变量总是能够得到优先赋值。

另外，JVM保证了`<clinit>()`方法在多线程的执行环境下的同步语义，即`<clinit>()`方法中的代码在多线程并发的环境下只会被某一个线程执行且只会被执行这一次，也就是说不会重复的执行`<clinit>()`方法(初始化静态变量和静态语句块)。

## 四、一个例子

在上述类加载机制的基础上分析以下代码会输出什么

```
public class Singleton {

    private static int x = 0;
    private static int y;
    private static Singleton instance = new Singleton();
    
    private Singleton() {
        x++;
        y++;
    }
    
    public static Singleton getInstance() {
        return instance;
    }
    
    public static void main(String[] args) {
        Singleton singleton = Singleton.getInstance();
        System.out.println(singleton.x);
        System.out.println(singleton.y);
    }

}
```

我们重点关注准备和初始化阶段，因为这是最重要的我们可以直观感受到的阶段

首先在准备过程中，类变量被赋予默认值，如下

```
x = 0;
y = 0;
instance = null;
```

然后在初始化阶段中执行`<clinit>()`方法，类变量被赋予正确值，如下

```
x = 0;
y = 0;
instance = new Singleton();
```

在`new Singleton()`的时候会执行类的构造函数，而在构造函数中分别对x和y进行了自增操作，所以结果为

```
x = 1;
y = 1;
```

现在调换一下`private static Singleton instance = new Singleton()`的顺序，如下

```
public class Singleton {

    private static Singleton instance = new Singleton();
    private static int x = 0;
    private static int y;
    
    private Singleton() {
        x++;
        y++;
    }
    
    public static Singleton getInstance() {
        return instance;
    }
    
    public static void main(String[] args) {
        Singleton singleton = Singleton.getInstance();
        System.out.println(singleton.x);
        System.out.println(singleton.y);
    }

}
```

同样的在准备阶段，结果为

```
instance = null;
x = 0;
y = 0;
```

初始化阶段中，在`new Singleton()`的时候会执行类的构造函数，执行结果为

```
x = 1;
y = 1;
```

然后顺序继续初始化静态变量x，由于x没有显式地进行赋值，因此0才是所期望的正确赋值，再继续初始化静态变量y，由于y没有给定初始值，所在在构造函数中计算所得的值就是所谓的正确赋值，最终结果为

```
x = 0;
y = 1;
```