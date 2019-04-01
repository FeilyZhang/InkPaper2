title: "再看封装、继承和多态"
date: 2018-12-14 15:37:04 +0800
update: 2018-12-14 15:37:04 +0800
author: me
cover: "-/images/java.jpg"
tags:
    - Java Core
preview: 封装是对成员变量的包装和隐藏；继承指的是子类可以继承父类的非私有属性和方法，在此之外，子类可以定义自己的和父类无关的属性和方法；多态是基于继承的。

---

封装是对成员变量的包装和隐藏。实现封装首先要将被封装的属性设置为私有private，然后对其提供相应的set/get方法进行特定的访问，set方法之外的方法只能通过get方法读取到被private修饰的变量的值而无法修改，且只能通过get方法获取，这就是封装。通过一个例子说明一下

```
package springTest;

public class People {

    private String name;
    private int age;
    private String sex;
    
    public void setName(String name) {
        this.name = name;
    }
    
    public void setAge(int age) {
        this.age = age;
    }
    
    public void setSex(String sex) {
        this.sex = sex;
    }
    
    public String getName() {
        return name;
    }
    
    public int getAge() {
        return age;
    }
    
    public String getSex() {
        return sex;
    }
    
}
package springTest;

public class PeopleMain {

    public static void main(String[] args) {
        People people = new People();
        // 只能通过set方法设置私有属性
        people.setName("Feily Zhang");
        people.setAge(21);
        people.setSex("man");
        // System.out.println(people.name + " " + people.age + " " + people.sex);  错误，私有属性只能通过get方法获取
        System.out.println(people.getName() + " " + people.getAge() + " " + people.getSex());
    }
}
```

继承指的是子类可以继承父类的非私有属性和方法，在此之外，子类可以定义自己的和父类无关的属性和方法，换句话说就是既然已经有类有这个方法，而我正好要用这个方法，那么我就不用再写，直接extends，我自己就可以获得这个方法。

需要特别注意的是，父类一定会有默认构造方法(即使没写那么编译期间也会填充一个)，子类可以在自己的构造方法(任意构造方法不一定是默认构造器)通过super()初始化父类构造器也可以不初始化交给编译器自动完成；但是如果父类有带参数的非默认构造器，那么子类必须在自己的构造器内通过super(参数)初始化父类，否则会报错。

```
public class Animal {

    private String animalName;
    
    public void setAnimalName(String animalName) {
        this.animalName = animalName;
    }
    
    public String getAnimalName() {
        return animalName;
    }
    
    public void say() {
        System.out.println(animalName + " , I am saying!");
    }
    
}
```

以上定义了一个Animal类，并封装了私有属性animalName，使得只能通过set和get方法来访问，我们接下来定义一个Dog类，该类extends了Animal类，那么只要其继承了Animal，就获得了Animal私有属性之外的所有属性和方法，如下

```
package springTest;

public class Dog extends Animal{

}
```

我们什么也不用写，因为通过extends已经获得了Animal的私有animalName之外所有属性和方法

然后验证一下

```
package springTest;

public class AnimalMain {

    public static void main(String[] args) {
        Dog dog = new Dog();
        dog.setAnimalName("Dog");
        dog.say();
    }

}
```

输出为

```
Dog , I am saying!
```

如果我们不用继承的方式，那么等同于如下代码

```
package springTest;

public class Dog{

    private String dogName;
    
    public void setDogName(String dogName) {
        this.dogName = dogName;
    }
    
    public String getDogName() {
        return dogName;
    }
    
    public void say() {
        System.out.println(dogName + " , I am saying!");
    }
}

package springTest;

public class AnimalMain {

    public static void main(String[] args) {
        Dog dog = new Dog();
        dog.setDogName("Dog");
        dog.say();
    }

}
```

在我们已经有Animal的情况下，是不是就没有必要写Dog的重复代码？所以通过继承可以简化冗余代码。

总体来说，继承还是比较常规的，就是子类没有必要写父类拥有的代码，直接拿来用就行了。但是，关于继承，还有如下注意点

第一、关于构造方法

父类一定会有默认构造方法(即使没写那么编译期间也会填充一个)，子类可以在自己的构造方法(任意构造方法不一定是默认构造器)通过super()初始化父类构造器也可以不初始化交给编译器自动完成；但是如果父类有带参数的非默认构造器，那么子类必须在自己的构造器内通过super(参数)初始化父类，否则会报错。

在上面的例子中，父类没有写默认构造器，但是编译期间填充了一个，子类也没有构造器，但是运行成功了。这是因为父类的默认构造器子类不需要显式super，所以有没有构造器都无所谓。那我们改动一下。让父类拥有非默认构造器

```
package springTest;

public class Animal {

    private String animalName;
    
    public Animal(String animalName) {
        this.animalName = animalName;
    }
    /*
    public void setAnimalName(String animalName) {
        this.animalName = animalName;
    }
    
    public String getAnimalName() {
        return animalName;
    }
    */
    public void say() {
        System.out.println(animalName + " , I am saying!");
    }
    
}

package springTest;

public class Dog extends Animal{
    /*
    private String dogName;
    
    public void setDogName(String dogName) {
        this.dogName = dogName;
    }
    
    public String getDogName() {
        return dogName;
    }
    
    public void say() {
        System.out.println(dogName + " , I am saying!");
    }
    */
}
```

那么Dog类一定会提示你

```
Implicit super constructor Animal() is undefined for default constructor. Must define an explicit constructor
```

所以加上构造方法，如下

```
package springTest;

public class Dog extends Animal{

    public Dog(String animalName) {
        super(animalName);
    }
    /*
    private String dogName;
    
    public void setDogName(String dogName) {
        this.dogName = dogName;
    }
    
    public String getDogName() {
        return dogName;
    }
    
    public void say() {
        System.out.println(dogName + " , I am saying!");
    }
    */
}
```

再测试一下

```
package springTest;

public class AnimalMain {

    public static void main(String[] args) {
        Dog dog = new Dog("Dog");;
        dog.say();	//Dog , I am saying!
    }

}

```

运行是没有问题的。所以，一定要注意。

第二、关于构造方法调用方法

构造方法调用方法应该尽可能避免，因为子类一旦重写这个方法，那么一旦子类重写的方法有变量参与，就会造成非预期结果，这是因为，在new子类的过程中是先初始化父类的，由于父类构造方法调用了方法，再由于子类重写了这个方法且有变量参与，那么由于该方法被重写，父类初始化的时候就调用重写后的方法，因为父类在子类之前被初始化，所以父类读取到的方法的变量就会是默认值(int为0).一个例子说明

```
package springTest;

public class Base {

    public Base() {
        test();
    }
    
    public void test() {
        
    }
}
```

父类Base的构造方法调用了test方法

```
package springTest;

public class Child extends Base {

    private int a = 111;
    public Child() {
        
    }
    public void test() {
        System.out.println(a);
    }
}
```

子类重写了test方法，导致new子类时先初始化父类，父类直接调用子类重写的test方法，由于子类还未new，那么a为0；等父类初始化之后，子类被new，那么调用test方法才会输出111

```
package springTest;

public class BaseMain {

    public static void main(String[] args) {
        Child c = new Child();	//0
        c.test();	//111
    }

}
```

但是为什么父类初始化的时候执行的是子类的test方法呢？怎么不执行父类自己的test？

这牵扯到动态绑定和静态绑定的概念，Java中实例变量、静态变量、静态方法、private方法都是静态绑定的，其余均是动态绑定。静态绑定的话如果子类对象向上转型为父类对象那么子类重写的父类方法无效即向上转型的子类对象执行的是父类的方法，如果不转型那么执行的仍然是子类的方法。动态绑定中无论子类对象是否转型，(不管是父类还是子类)执行的都是重写后的子类的方法。

第三、关于重名(重写)与静态绑定

上面其实已经说了，静态绑定的概念，我们先看一下动态绑定，仍然看上面的例子，只不过我们增加一个转型

```
package springTest;

public class BaseMain {

    public static void main(String[] args) {
        Child c = new Child();
        Base b = c;
        c.test();
        b.test();
    }

}
```

输出为

```
0
111
111
```

可以看出，执行的还是子类的方法

说个题外话，为什么只有一个0，不是应该有两个吗？这是因为Child c = new Child()的时候父类已经被初始化，子类也被初始化，那么自然a的值为111而非0.

再看一下静态绑定，如果我们子类重写父类的实例变量、静态变量、静态方法、private方法，那么如果转型就会调用父类的方法，不转型则是子类的方法，如下

```
package springTest;

public class Base {

    public String a = "base";
    public static String b = "static_base";
    
    public static void staticTest() {
        System.out.println("static_staticTest " + b);
    }
}
package springTest;

public class Child extends Base {

    public String a = "child";
    public static String b = "static_child";
    
    public static void staticTest() {
        System.out.println("static_staticTest_child " + b);
    }
}
package springTest;

public class BaseMain {

    public static void main(String[] args) {
        // 不转型，输出的就是子类的重写
        Child c = new Child();
        System.out.println(c.a);
        c.staticTest();
        //转型，输出的是父类的属性和方法
        Base b = c;
        System.out.println(b.a);
        b.staticTest();
    }

}
```

结果为

```
child
static_staticTest_child static_child
base
static_staticTest static_base
```

显而易见

第四、重载时调用父类还是子类方法

```
package springTest;

public class Base {
    public int sum(int a, int b) {
        System.out.println("base_int_int");
        return a + b;
    }
}
package springTest;

public class Child extends Base {

    public long sum(long a, long b) {
        System.out.println("child_long_long");
        return a + b;
    }
}
package springTest;

public class BaseMain {

    public static void main(String[] args) {
        Child c = new Child();
        int a = 1, b = 2;
        System.out.println(c.sum(a, b));
    }

}
```

结果为

```
base_int_int
3
```

这是因为重载时是根据数据类型匹配方法的。由于Main方法中a和b后市int型参数，父类的参数都是int型，所以使用父类的方法，如果我们把main方法中的a和b改为long型参数，那么就会调用子类的方法，如下

```
package springTest;

public class BaseMain {

    public static void main(String[] args) {
        Child c = new Child();
        long a = 1L, b = 2L;
        System.out.println(c.sum(a, b));
    }

}
```

结果为

```
child_long_long
3
```

第五、父子类型的转换

子类型转为父类型称为向上转型，我们上面有涉及，向下转型(父类转型为子类)呢？

语法上可以通过强制类型转换进行，但是不一定成功。

一个父类的变量能不能转型为一个子类的变量，取决于这个父类变量的动态类型是不是这个子类或者这个子类的子类。

比如

```
Base b = new Child();
Child c = (Child) b;
```

这是可以成功的，因为父类Base的动态类型就是子类Child，所以可以成功

但是

```
Base b = new Base();
Child c = (Base) b;
```

这是会失败的，因为Base的动态类型不确定是不是Child

另外，可以通过instanceof关键字查看父类能否转型为子类，如下

```
public boolean canCast(Base b) {
    return b instanceof Child;
}
```

第六、可见性重写

重写方法一般情况下不会修改父类方法的可见性，但是需要注意的是子类不能降低父类方法的可见性。

可见性由高到低为：public、protected、private。

第七、父类的final方法不能被继承

final关键字可以修饰类、变量、方法，不能修饰接口，因为接口必须被实现才有意义，一旦声明为final就无法被其他类实现。

final修饰的类不能被继承、修饰的方法和变量不能被重写。

最后，通过一个例子来说明，继承的真谛：利用继承的特性，只写有必要的代码，没有必要且已经存在的代码直接继承调用即可。

假定现在我们已经有了这个类

```
package springTest;

public class Base {
    protected int currentStep;
    protected void step1() {
        
    }
    protected void step2() {
        
    }
    protected void action() {
        this.currentStep = 1;
        step1();
        this.currentStep = 2;
        step2();
    }
}
```

我们需要的是action里面的功能，发现完全满足，但是step1和step2并没有被实现，那么简单我们实现了这两个方法然后调用action即可。子类如下

```
package springTest;

public class Child extends Base {

    public void step1() {
        System.out.println("Child step " + this.currentStep);
    }
    
    public void step2() {
        System.out.println("Child step " + this.currentStep);
    }
}
```

我们升级了父类方法的可见性，然后调用了父类的实例变量，请注意，我们并没有重写父类的实例变量，调用完全是可以的，那么我们不管是否向上转型，那么我们都会得到相同的结果

```
package springTest;

public class BaseMain {

    public static void main(String[] args) {
        Child c = new Child();
        c.action();
        Base b = c;
        b.action();
    }

}
```

结果为

```
Child step 1
Child step 2
Child step 1
Child step 2
```

多态

多态是基于继承的，多态存在的3个条件是

+ 子类继承父类； 
+ 子类重写父类的方法； 
+ 父类引用指向子类对象

就用之前的Animal举个例子

```
package springTest;

public class Animal {
    public void say() {
    }
}
```

Animal我们定义一个空方法，类似于接口和抽象类，多态我们一般都是这么做

再定义Cat和Dog，分别继承Animal，这是多态的第一个条件，然后重写say方法，这是第二个条件

```
package springTest;

public class Dog extends Animal{

    String dogName;
    
    public Dog(String dogName) {
        this.dogName = dogName;
    }
    public void say() {
        System.out.println("I am a " + dogName);
    }

}
package springTest;

public class Cat extends Animal{

    String catName;
    
    public Cat(String catName) {
        this.catName = catName;
    }
    
    public void say() {
        System.out.println("I am a " + catName);
    }
}
```

主文件中，向上转型为Animal，这是多态的第三个条件，即父类引用指向子类对象，如下

```
package springTest;

public class AnimalMain {

    public static void main(String[] args) {
        Animal dog = new Dog("dog");
        dog.say();
        Animal cat = new Cat("cat");
        cat.say();
    }

}
```

输出为

```
I am a dog
I am a cat
```

好吧，你可能说也没见多态有什么威力呀，不用转型照样可以实现需求啊。主要是这样更加规范和便于理解

改造一下，我们再抽象一个动物管理员出来

```
package springTest;

public class AnimalManager {

    private static final int MAX_NUM = 100;
    private Animal[] animals = new Animal[MAX_NUM];
    private int animalNum = 0;
    
    public void addAnimal(Animal animal) {
        if (animalNum < MAX_NUM) { 
            animals[animalNum++] = animal; 
        } 
    } 
	
    public void say() { 
        for(Animal animal : animals) { 
            animal.say(); 
        }
    }
}
```

相应的主文件为

```
package springTest;

public class AnimalMain {

    public static void main(String[] args) {
        AnimalManager am = new AnimalManager();
        am.addAnimal(new Dog("dog"));
        am.addAnimal(new Cat("cat"));
        am.say();
    }

}
```

便于管理，可拓展性极强。

变量Animal可以引用任何Animal子类类型的对象，这叫多态，即一种类型的变量，可以引用多种实际类型对象。这样，对于类型Animal，我们称之位animal的静态类型，而Dog、Cat等，我们称之为animal的动态类型，AnimalManager的say方法调用的是其对应的动态类型(如Dog、Cat)的say方法，这称之为动态绑定。

多态和动态绑定是计算机程序的一种重要的思维方式，使得操作对象的程序不需要关注对象的实际类型，因为被父类统一起来了，这样就可以同意处理不同的对象，而且能够实现每个对象的特有行为。

总而言之，多态就是同一个类型在不同的指向下有不同的状态，所谓状态就是特定的属性和行为。