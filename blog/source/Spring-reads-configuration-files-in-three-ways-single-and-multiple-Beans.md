title: "Spring中读取配置文件的三种方式及单例多例Bean"
date: 2018-12-12 19:35:30 +0800
update: 2018-12-12 19:35:30 +0800
author: me
cover: "-/images/Spring-Framework.jpg"
tags:
    - Spring Framework
preview: Spring中读取配置文件的三种方式。

---

Spring中读取配置文件的三种方式
```
Resource res = new ClassPathResource("packageName/bean.xml");
BeanFactory factory = new XmlBeanFactory(res);    //通过XmlBeanFactory类读取

ApplicationContext factory = new ClassPathXmlApplicationContext("packageName/bean.xml");    //通过ClassPathXmlApplicationContext类读取

ApplicationContext factory = new FileSystemXmlApplicationContext(new String[] {"bean.xml的绝对路径"})
或者
ApplicationContext factory = new FileSystemXmlApplicationContext("bean.xml的绝对路径")    //通过FileSystemXmlApplicationContext类读取
```
单例Bean就是Main文件在运行中只会得到一个Bean实例，即每次得到的Bean实例是相同的，也称无状态Bean，一般把公用的资源可以保存为单例Bean；

多例Bean就是Main文件在运行中每getBean()都会得到一个截然不同的Bean实例，也称有状态Bean，那么就可以用有状态Bean来保存比较私有的资源。

单例和多例Bean可以在配置文件中的bean标签中设置，需要特别注意的是，旧版Spring中是通过singleton属性来设置的，当值为true时为单例无状态Bean，当值为false时为多例有状态Bean；在新版Spring中，是通过scope属性来设置的，当值为prototype时为有状态多例Bean，当值为singleton时为单例无状态Bean。

一个简单的例子来说明单例和多例Bean

这是一个提供随机数的类
```
package springTest;

public class Randoms {
    private int i  = (int) (100 * Math.random());
    public void printRandom() {
        System.out.println(i);
    }
}
```
配置文件设置其为多例Bean，由于和其它类没有依赖关系，所以只需要定义bean即可，无需定义其他的属性
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
<bean id="ran" class="springTest.Randoms" scope="prototype"/>
</beans>
package springTest;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class SpringRandoms {

    public static void main(String[] args) {
        ApplicationContext factory = new ClassPathXmlApplicationContext("springTest/RandomsBean.xml");
        Randoms r1 = (Randoms)factory.getBean("ran");
        Randoms r2 = (Randoms)factory.getBean("ran");
        System.out.println(r1 == r2);
        r1.printRandom();
        r2.printRandom();
    }

}
```
运行结果为
```
false
28
15
```
通过配置文件将其设置为单例Bean，如下
```
<bean id="ran" class="springTest.Randoms" scope="singleton"/>
```
那么运行结果为
```
true
93
93
```