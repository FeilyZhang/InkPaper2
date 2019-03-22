title: "通过@Autowired注解进行Spring装配"
date: 2018-12-12 20:28:54 +0800
update: 2018-12-12 20:28:54 +0800
author: me
cover: "-/images/Spring-Framework.jpg"
tags:
    - Spring Framework
preview: Autowired按byType自动注入，而Resource默认按 byName自动注入。

---

就是变动一下配置文件，改为
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
<bean id="helloWorldSpring" class="HelloWorldSpring"/>    <!-- 注意注入类的id可以随意定义，无需与被注入类字段保持一致 ->
<bean id="sayHello" class="SayHello" />
<context:annotation-config />    <!-- 注意这里的代码 -->
</beans>
```
再在被注入类字段上进行注解
```
import org.springframework.beans.factory.annotation.Autowired;

public class SayHello {
    
    @Autowired
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
```
这种注入方式等同于autowire="byType"。只不过尽可能少用这种方式，缺点如下

+ 代码可读性差，不容易维护，因为类之间的依赖关系我们只能在代码中找
+ 通用性不好，如果我们哪天不用Spring了，那么我们就得一个个删除注解。