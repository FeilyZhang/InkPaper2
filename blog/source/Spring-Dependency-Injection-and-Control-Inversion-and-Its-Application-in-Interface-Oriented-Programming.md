title: "Spring依赖注入(DI)与控制反转(IoC)及在面向接口编程上的应用"
date: 2018-12-12 17:24:58 +0800
update: 2018-12-12 17:24:58 +0800
author: me
cover: "-/images/Spring-Framework.jpg"
tags:
    - Spring Framework
preview: Spring依赖注入，指的是类之间对象实例的传递方式；将创建对象的控制权由自写代码交给Spring容器(或Spring jar包)，即控制反转。

---

所谓Spring依赖注入，指的是类之间对象实例的传递方式。传统的类间对象(或参数)传递是通过实例化一个类，然后将该对象传递给另一个类的方法,而依赖注入则是通过配置文件将一个类的实例直接传递给另一个类的变量或者方法，根本不用显式的new对象然后再作为参数传递。如下
```
public class SayHello {
    private HelloWorldSpring helloWorldSpring;
    public HelloWorldSpring getHelloWorldSpring() {
        return helloWorldSpring;
    }
    public void setHelloWorldSpring(HelloWorldSpring helloWorldSpring) {
        this.helloWorldSpring = helloWorldSpring;
    }
    public void sayHello() {
        System.out.println("Say Hello:" + helloWorldSpring.sayHello());
    }
}
public class HelloWorldSpring {
    private String sayContent;
    public String sayHello() {
        System.out.println("HelloWorld Spring!");
        return "HelloWorld Spring";
    }
}
public class HelloMain {

    public static void main(String[] args) {
        SayHello sayHello = new SayHello();
        // new实例然后传递给调用者，耦合度高
        sayHello.setHelloWorldSpring(new HelloWorldSpring());
        sayHello.sayHello();
    }

}
```
以上HelloMain是传统的注入方式，明显耦合度高，但是依赖注入可不这么干，是通过配置文件来实现注入，如下
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:util="http://www.springframework.org/schema/util"
    xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.1.xsd
        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.3.xsd
        http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util-4.3.xsd">
<bean id="helloWorldSpring" class="HelloWorldSpring"/>
<bean id="sayHello" class="SayHello">
    <property name="helloWorldSpring" ref="helloWorldSpring"/>
</bean>
</beans>
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class SpringMain {
    public static void main(String[] args) {
        ApplicationContext factory = new ClassPathXmlApplicationContext("bean.xml");
        SayHello sayHello = (SayHello) factory.getBean("sayHello");
        sayHello.sayHello();
    }
}
```
`ApplicationContext factory = new ClassPathXmlApplicationContext("bean.xml")`读取配置文件，然后通过getBean()方法获取一个目标为sayHello的对象，其实就是通过配置文件实例化了SayHello类，sayHello对应bean.xml文件里的SayHello类的id。

特别注意的是此处代码并没有在包中(一个空包，即文件头没有package声明)，所以bean文件中bean标签的class属性并没有加包名前缀，同理SpringMain读取配置文件也没有加包名路径。

以上就是Spring依赖注入。

至于Spring控制反转，则说的是类的实例化不用通过new显式的实现，而是将创建对象的控制权由自写代码交给Spring容器(或Spring jar包)，即控制反转。上述SpringMain可以明显的看出，sayHello对象的创建是通过`factory.getBean()`方法实现的，而不是new操作符。

简言之，控制反转说的是类的初始化(创建)不用new而交给Spring容器，依赖注入说的是通过配置文件实现类间的对象传递。其目的就是解耦.

再通过面向接口编程深入理解IoC与DI

面向接口编程的本质是缩小修改的影响范围,先看一个实际的例子，有一个提供随机数生成的类，有一个专门使用随机数的类，如果用常规思维来实现的话，无非就是如下的代码
```
package springTest;

public class Random {
    private int randomNum = (int) (50 * Math.random());
    public void printRandom() {
        System.out.println(randomNum);
    }
}
package springTest;

public class TestMain {
    public static void main(String[] args) {
        new Random().printRandom();
    }
}
```
而面向接口编程则是如下的代码

先是接口，向外部调用者提供用于返回Random对象的createRandom方法
```
package springTest;

public interface HelloInterface {
    public abstract Random createRandom();
}
```
再是接口的实现，外部调用者通过createRandom得到Random对象时，是直接和接口打交道，根本不知道有该实现类的存在
```
package springTest;

public abstract class HelloAbstract implements HelloInterface {
    @SuppressWarnings("unused")
    private Random random;
    public void setRandom(Random random) {
        this.random = random;
    }
    public abstract Random createRandom();
}
```
实际上，该实现类就是在配置文件中直接将Random类注入到其中，由于实现类实现了接口，而实现类被注入了Random类，那么实现类就可以当作接口与Random类之间的桥梁，这个桥梁是通过配置文件依赖注入建造的。

Random类如下
```
package springTest;

public class Random {
    private int randomNum = (int) (50 * Math.random());
    public void printRandom() {
        System.out.println(randomNum);
    }
}
```
SpringMain如下
```
package springTest;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class SpringMain {
    @SuppressWarnings("resource")
    public static void main(String[] args) {
        ApplicationContext factory = new ClassPathXmlApplicationContext("springTest/bean.xml");
        HelloInterface hi = (HelloInterface) factory.getBean("hello");
        Random random = hi.createRandom();
        random.printRandom();
    }

}
```
配置文件为
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:util="http://www.springframework.org/schema/util"
    xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.1.xsd
        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.3.xsd
        http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util-4.3.xsd">
<bean id="ran" class="springTest.Random"/>
<bean id="hello" class="springTest.HelloAbstract">
    <lookup-method name="createRandom" bean="ran"/>
</bean>
</beans>
```
可见，是通过lookup-method标签将id为ran的bean注入到id为hello的bean的createRandom方法中，即createRandom方法的返回值就是这个id为ran的bean(对象)。