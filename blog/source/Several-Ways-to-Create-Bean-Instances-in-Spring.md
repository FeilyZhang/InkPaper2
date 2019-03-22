title: "Spring中创建Bean实例的几种方式"
date: 2018-12-13 10:13:59 +0800
update: 2018-12-13 10:13:59 +0800
author: me
cover: "-/images/Spring-Framework.jpg"
tags:
    - Spring Framework
preview: 无参默认构造器、有参自定义构造器、有参通过Setter方式创建。

---

第一种方式，(无参创建)通过无参构造器创建一个bean实例
```
package springTest;

public class Product {
	/*
    public Product() {
        
    }*/
    public void printProduct1() {
        System.out.println("Hello, I am product1!");
    }

    public void printProduct2() {
        System.out.println("Hello, I am product2!");
    }
}
```
其中默认构造器可以省略，即使没有那么Java类在编译期间也会自动填充一个默认构造器

然后是配置文件
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
<bean name="pro" class="springTest.Product"/>
</beans>
```
再是主方法类
```
ackage springTest;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class ProductMain {

    public static void main(String[] args) {
        ApplicationContext factory = new ClassPathXmlApplicationContext("springTest/productBean.xml");
        Product product = (Product) factory.getBean("pro", Product.class);
        product.printProduct1();
        product.printProduct2();
    }
}
```
运行结果为
```
Hello, I am product1!
Hello, I am product2!
```
这里并不会牵扯到依赖注入，因为只是一个类，并没有类之间的依赖关系，所以和正常的Java类一样，不需要编写依赖注入时被注入类的set/get方法。

但是如果类有构造器而且不是默认构造器，该怎么办？用第一种方法是会报错的，如下
```
Exception in thread "main" org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'pro' defined in class path resource [springTest/productBean.xml]: Instantiation of bean failed; nested exception is org.springframework.beans.BeanInstantiationException: Could not instantiate bean class [springTest.Product]: No default constructor found; nested exception is java.lang.NoSuchMethodException: springTest.Product.()
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.instantiateBean(AbstractAutowireCapableBeanFactory.java:1076)
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBeanInstance(AbstractAutowireCapableBeanFactory.java:1021)
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:504)
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:475)
	at org.springframework.beans.factory.support.AbstractBeanFactory$1.getObject(AbstractBeanFactory.java:302)
	at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:228)
	at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:298)
	at org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:193)
	at org.springframework.beans.factory.support.DefaultListableBeanFactory.preInstantiateSingletons(DefaultListableBeanFactory.java:703)
	at org.springframework.context.support.AbstractApplicationContext.finishBeanFactoryInitialization(AbstractApplicationContext.java:760)
	at org.springframework.context.support.AbstractApplicationContext.refresh(AbstractApplicationContext.java:482)
	at org.springframework.context.support.ClassPathXmlApplicationContext.(ClassPathXmlApplicationContext.java:139)
	at org.springframework.context.support.ClassPathXmlApplicationContext.(ClassPathXmlApplicationContext.java:83)
	at springTest.ProductMain.main(ProductMain.java:9)
Caused by: org.springframework.beans.BeanInstantiationException: Could not instantiate bean class [springTest.Product]: No default constructor found; nested exception is java.lang.NoSuchMethodException: springTest.Product.()
	at org.springframework.beans.factory.support.SimpleInstantiationStrategy.instantiate(SimpleInstantiationStrategy.java:85)
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.instantiateBean(AbstractAutowireCapableBeanFactory.java:1069)
	... 13 more
Caused by: java.lang.NoSuchMethodException: springTest.Product.()
	at java.lang.Class.getConstructor0(Unknown Source)
	at java.lang.Class.getDeclaredConstructor(Unknown Source)
	at org.springframework.beans.factory.support.SimpleInstantiationStrategy.instantiate(SimpleInstantiationStrategy.java:80)
	... 14 more
```
这种情况下第二种方法就显得很有必要了

第二种方法，(有参通过构造器创建)通过配置文件向构造器传递参数实例化bean

我们先将Product类改为有构造器参与的形式
```
package springTest;

public class Product {
    
    String product1;
    String product2;
    
    public Product(String product1, String product2) {
        this.product1 = product1;
        this.product2 = product2;
    }
    
    public void printProduct1() {
        System.out.println("Hello, I am " + product1);
    }

    public void printProduct2() {
        System.out.println("Hello, I am " + product2);
    } 
}
```
再通过配置文件传递参数实例化bean(其实就是实例化类)
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
<bean name="pro" class="springTest.Product">
    <constructor-arg index="0" value="product1"/>
    <constructor-arg index="1" value="product2"/>
</bean>
</beans>
```
以上配置文件的bean标签的constructor-arg标签的index属性等同于name属性，如下
```
<bean name="pro" class="springTest.Product">
    <constructor-arg name="product1" value="product1"/>
    <constructor-arg name="product2" value="product2"/>
</bean>
```
是同样的功能

再运行之
```
package springTest;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class ProductMain {

    public static void main(String[] args) {
        ApplicationContext factory = new ClassPathXmlApplicationContext("springTest/productBean.xml");
        Product product = (Product) factory.getBean("pro", Product.class);
        product.printProduct1();
        product.printProduct2();
    }
}
```
结果为
```
Hello, I am product1
Hello, I am product2
```
当然这只是向一个bean传递参数的一种方式，还有一种方式是通过Setter方式

第三种，(有参通过Setter方式创建)通过Setter方式传递参数给bean创建其实例

这种方法自然要对相应的字段编写对应的get/set方法，如下
```
package springTest;

public class Product {
    
    String product1;
    String product2;
    /*
    public Product(String product1, String product2) {
        this.product1 = product1;
        this.product2 = product2;
    }
    */
    public String getProduct1() {
        return product1;
    }
    
    public String getProduct2() {
        return product2;
    }
    
    public void setProduct1(String product1) {
        this.product1 = product1;
    }
    
    public void setProduct2(String product2) {
        this.product2 = product2;
    }
    
    public void printProduct1() {
        System.out.println("Hello, I am " + product1);
    }

    public void printProduct2() {
        System.out.println("Hello, I am " + product2);
    } 
}
```
配置文件只需要把constructor-arg标签改为property即可，如下
```
<bean name="pro" class="springTest.Product">
    <property name="product1" value="product1"/>
    <property name="product2" value="product2"/>
</bean>
```
主文件不做改动，仍然是同样的运行结果

这种情况下你可以注释掉构造器，前提是你没有使用constructor-arg标签传递参数。当然，你也可以二者都使用，如下
```
package springTest;

public class Product {
    
    String product1;
    String product2;
    
    public Product(String product1) {
        this.product1 = product1;
    }
    
    public String getProduct2() {
        return product2;
    }
    public void setProduct2(String product2) {
        this.product2 = product2;
    }
    
    public void printProduct1() {
        System.out.println("Hello, I am " + product1);
    }

    public void printProduct2() {
        System.out.println("Hello, I am " + product2);
    } 
}
<bean name="pro" class="springTest.Product">
    <constructor-arg name="product1" value="product1"/>
    <property name="product2" value="product2"/>
</bean>
package springTest;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class ProductMain {

    public static void main(String[] args) {
        ApplicationContext factory = new ClassPathXmlApplicationContext("springTest/productBean.xml");
        Product product = (Product) factory.getBean("pro", Product.class);
        product.printProduct1();
        product.printProduct2();
    }
}
```
此外，这里配置文件的属性用的是name而不是id，这两者都可以用而且主文件中`factory.getBean("pro", Product.class)`与`factory.getBean("pro")`是同样的功能，可以自由组合。