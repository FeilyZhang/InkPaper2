title: "Spring注入的另外三种常用方式与AutoWire"
date: 2018-12-12 20:10:08 +0800
update: 2018-12-12 20:10:08 +0800
author: me
cover: "-/images/Spring-Framework.jpg"
tags:
    - Spring Framework
preview: Spring的注入方式有六种，比较常用的有三种自动装配。

---

先说明，以下是公用代码，只是配置文件不同
```
public class HelloWorldSpring {
    private String sayContent;
    public String sayHello() {
        System.out.println("HelloWorld Spring!");
        return "HelloWorld Spring";
    }
}

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
Spring的注入方式有六种，比较常用的有三种自动装配

第一种自动装配，当被注入类配置文件bean标签中属性autowire的取值为no时，表明不使用自动装配，即必须显式的使用<ref>标签明确的指定注入的bean
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
<bean id="sayHello" class="SayHello" autowire="no">    <!-- 请注意这里的autowire属性及其值 -->
    <property name="helloWorldSpring" ref="helloWorldSpring"/>    <!-- 还有这里的property标签 -->
</bean>
</beans>
```
可见，这种方式无异于脱了裤子放屁，直接去掉autowire属性即可，就和前几篇文章用的注入方式一致

第二种自动装配，当被注入类配置文件bean标签中属性autowire的取值为byName时，表明根据属性名自动装配，那么将会检查Spring容器，根据名字查找属性完全一致的Bean然后进行注入。简言之就是与被注入类中字段名称相同的配置文件中的bean标签的id值代表的class类对象将会被装配(注入)，如下
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
<bean id="sayHello" class="SayHello" autowire="byName">    <!-- 请注意这里的autowire属性及其值 -->
</bean>
</beans>
```
第三种自动装配，当被注入类配置文件bean标签中属性autowire的取值为byType时，表明按类型自动装配，即被注入类字段的数据类型(为类)与配置文件中bean标签的注入类class属性的类型名一致时，才会装配，如下
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
<bean id="sayHello" class="SayHello" autowire="byType">    <!-- 请注意这里的autowire属性及其值 -->
</bean>
</beans>
```
需要特别注意的是，被注入类中的相关字段一定要有对应的set方法(和get方法)，不然byName和byType会报错NPE,no和以前的那种装配方式则会报错如下
```
Caused by: org.springframework.beans.NotWritablePropertyException: Invalid property 'helloWorldSpring' of bean class [SayHello]: Bean property 'helloWorldSpring' is not writable or has an invalid setter method. Does the parameter type of the setter match the return type of the getter?
```