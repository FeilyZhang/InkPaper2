title: "再看接口、抽象类与内部类"
date: 2018-12-14 17:34:30 +0800
update: 2018-12-14 17:34:30 +0800
author: me
cover: "-/images/java.jpg"
tags:
    - Java Core
preview: 接口屏蔽了类型的差异性，只关注能力或者动作; 而抽象类与接口的区别在于抽象类抽象实体，而接口只抽象动作。内部类与包含它的外部类有着密切的关系，而与其他类关系不大，定义在类内部，可以实现对外的完全隐藏，代码上更为紧凑和简洁。

---

接口屏蔽了类型的差异性，只关注能力或者动作。比如说计算机USB接口，它只是定义了一个规则，并没有具体的实现(U盘)，但是却可以使得U盘与计算机交互，接口就是具备这样一种能力。

简单来说接口的使用包括三方，一方是使用者、一方是接口，另一方是接口的实现。接口实现者实现了接口定义的规则，而使用者通过接口使用实现者提供的服务，这就是他们之间的关系。这样就会降低耦合性，把各部分独立出来。

比如，我们以前的随机数服务为例，如下

```
public interface HelloInterface {
    public abstract Random createRandom();
}
```

然后我们定义接口的实现者

```
public class RealizeInterface implements HelloInterface {

    @Override
    public Random createRandom() {
        return new Random();
    }

}
```

该实现者返回一个随机数类，该类的定义如下

```
public class Random {
    private int randomNum = (int) (50 * Math.random());
    public void printRandom() {
        System.out.println(randomNum);
    }
}
```

接口的使用者为

```
public class Using {

    public static void main(String[] args) {
        HelloInterface hello = new RealizeInterface();
        Random random = hello.createRandom();
        random.printRandom();
    }
}
```

这就是接口的使用，也是面向接口编程，即我们所需要的服务全部是向接口索取，而非Random实体类，由于接口屏蔽了随机数的实现细节，那么我们获取到的随机数服务就是稳定的。

这里的例子是IoC与DI的面向接口编程的例子，只是没有使用依赖注入与控制反转。

下面说接口的一些细节

第一、接口中可以定义变量

变量的默认修饰符是public static final，引用方式为接口名.变量名。

第二、接口可以继承其他接口

接口可以继承其他接口，且可以是多继承，同样适用extends关键字

第三、instanceof

也可以使用instanceof判断某对象是否实现了某接口，使用方式为

对象 instanceof 接口
返回值是boolean型变量

第四、Java8对接口的增强

Java8中对接口进行了功能增强，使得接口可以定义静态方法和默认方法且都有实现体，金泰方法通过接口名调用，默认方法通过接口型的对象调用，且静态方法和默认方法都必须是public的

以随机数这个例子说明之，改动接口的代码，添加默认方法和静态方法，如下

```
public interface HelloInterface {
    public abstract Random createRandom();
    
    public static void printmMathRandom() {
        System.out.println(Math.random());
    }
    
    public default void printRandomClassRandom() {
        System.out.println(new java.util.Random().nextInt(10));
    }
}
```

主函数改一下

```
public class Using {

    public static void main(String[] args) {
        HelloInterface hello = new RealizeInterface();
        Random random = hello.createRandom();
        random.printRandom();
        hello.printRandomClassRandom();
        HelloInterface.printmMathRandom();
    }
}
```

输出为

```
16
6
0.31667981164852066
```

第五、Java9对接口的增强

Java9去掉了接口的静态方法和默认方法是public的限制，可以是private，这样就便于同一个接口内的其余方法引用private修饰的方法代码。由于我是Java8,因此，这里无法演示。

总结一下，面向接口编程是一种重要的程序思维方式，这种方式不仅可以复用代码，还可以降低耦合，提高灵活性，所以适合对大问题分而治之，与面向切面编程各有侧重。

抽象类

抽象类与具体类相对立又同一，抽象类只是无法创建对象，表达的是抽象的概念，除此之外与具体类并无差别，是用来继承的，也可以不继承直接通过main方法运行(一定是自身的main方法，因为因为无法创建对象，就无法在别的类的main方法中运行)。类因为抽象方法而抽象，没有抽象方法的抽象类是没意义的但是并不是错误的，也就是说抽象类中要有抽象方法(也可以没有)，但是抽象方法所在的类必须是抽象的，抽象类可以有非抽象方法的实现，抽象方法不能有方法体，可以有main方法，如下，我们定义一个计算图形面积和周长的抽象类

我们先演示一下抽象类可以有main方法，但不能实例化，如果我们要运行抽象类的非抽象方法，那么就只能写为静态方法，如下

```
public abstract class Shape {

    public static float length = 10f;
    public static float width = 20f;
    
    public static void area() {
        System.out.println(length * width);
    }
    
    public static void main(String[] args) {
        area();    //200.0
    }
}
```

当然，这不是我们的重点，我们直接定义一个抽象类

```
public abstract class Shape {
    public abstract void calArea();
    public abstract void calGirth();
}
```

看起开好像与接口无异，你是正确的，现在确实是这样，我们还得继续，再声明类继承抽象类并直接main运行

```
public class Square extends Shape {

    float length, width;
    
    public Square(float length, float width) {
        this.length = length;
        this.width = width;
    }
    public void calArea() {
        System.out.println(length * width);
    }

    public void calGirth() {
        System.out.println((length + width) * 2);
    }

    public static void main(String[] args) {
        Square s = new Square(10.0f, 20.0f);
        s.calArea();	//200.0
        s.calGirth();	//60.0
    }
}
```

看来确实与接口无异啊，只不过就是比接口高级一点，可以有默认方法实现(Java8中接口也有了)，可以有main方法(这个接口没有)。

其实，接口与抽象类还是有去别的，抽象类抽象实体，而接口只抽象动作。

还有二者其实是配合使用的，接口声明能力，抽象类提供默认实现(实现部分或者全部方法)，所以一个接口常常有一个抽象类相对应。那么逻辑现在就变成了，抽象类实现接口，原先的接口实现者现在继承抽象类，然后重写必要的方法供使用者使用，这是因为抽象类无法new对象，因此只能通过继承了抽象类的类来new对象，供接口使用者使用。

内部类与包含它的外部类有着密切的关系，而与其他类关系不大，定义在类内部，可以实现对外的完全隐藏，代码上更为紧凑和简洁。不过，内部类，只是Java编译器的概念，在运行时每个类都会被编译为一个独立的类，拥有独立的字节码文件。

内部类共有四种类型，分别是

+ 静态内部类 
+ 成员内部类 
+ 方法内部类 
+ 匿名内部类

静态内部类带有static关键字，如下

```
public class Outer {
    private static int shared = 100;
    public static class StaticInner {
        public void innerMethod() {
            System.out.println("inner " + shared);
        }
    }
    public void test() {
        new StaticInner().innerMethod();
    }
}
```

静态内部类可以访问对应外部类的静态变量和方法，但不能访问实例变量和方法。静态内部类可以被其余外部类访问，访问方式是外部类.静态内部类的方式。

成员内部类

```
public class Outer {
    private int a = 100;
    public class Inner {
        public void innerMethod() {
            System.out.println("outer a " + a);
            Outer.this.action();
        }
    }
    private void action() {
        System.out.println("action");
    }
    public void test() {
        new Inner().innerMethod();
    }
}
```

与静态内部类的区别是少了static关键字，可以访问外部类的变量和方法，如果内部类方法与外部类方法重名那么可以通过外部类.this.外部类方法的形式访问外部类方法。

方法内部类

顾名思义，就是内部类定义在方法中，如下

```
public class Outer {
    private int a = 100;
    public void test(final int param) {
        final String str = "hello";
        class Inner {
            public void innerMethod() {
                System.out.println("outer a " + a);
                System.out.println("param " + param);
                System.out.println("local var " + str);
            }
        }
        new Inner().innerMethod();
    }
}
```

如果包含内部类的方法是非静态方法，那么方法内部类可以访问方法之外外部类之内的非静态变量和方法；

如果包含内部类的方法是静态方法，那么内部类只能访问外部类的静态变量和静态方法。

方法内部类还可以访问方法的参数和方法中的局部变量，在Java8之前，这两种变量必须被声明为final，Java8之后不再要求，但变量不能重新赋值否则会编译错误。

匿名内部类

其语法为

```
new 父类(参数列表) {
	//匿名内部类的实现部分
}
或者
new 父接口() {
	//匿名内部类的实现部分
}
```

例如

```
public class Outer {
    public static void main(String[] args) {
        Shape shape = new Shape() {
            float length = 10f;
            float width = 20f;
            @Override
            public void calArea() {
                System.out.println(length * width);
            }

            @Override
            public void calGirth() {
                System.out.println((length + width) * 2);
            }
            
        };
        shape.calArea();	//200.0
        shape.calGirth();	//60.0
    }
}
```

通过匿名类重写了抽象父类Shape的方法，那么由于重写方法使得Shape变成了形式上的抽象类实际上的抽象类，所以可以new对象，然后调用方法。