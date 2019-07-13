title: "Spring IoC容器的接口与实现类"
date: 2019-07-13 14:43:49 +0800
update: 2019-07-13 14:43:49 +0800
author: me
cover: "-/images/Spring-Framework.jpg"
tags:
    - Spring
preview: Spring通过IoC(控制反转)来管理对象间的依赖关系。Spring的IoC容器是一个轻量级容器，没有侵入性，不需要依赖容器的API，也不需要实现一些特殊的接口。

---

## 一、Spring IoC容器概述

Spring通过IoC(控制反转)来管理对象间的依赖关系。Spring的IoC容器是一个轻量级容器，没有侵入性，不需要依赖容器的API，也不需要实现一些特殊的接口。

Spring IoC容器是Spring框架的核心，容器将创建配置的所有对象，将他们通过注入的方式连接在一起，最后销毁所创建的对象，Spring对象管理对象的整个生命周期。在Spring容器中使用依赖注入(DI)来管理组成应用程序的类对象。这些对象被称为Spring Beans。

Spring提供了实现`BeanFactory`和`ApplicationContext`两种不同接口类型的IoC容器。

## 二、BeanFactory容器

该容器是简化的IoC容器，提供了最简单的对象管理和依赖注入的基本支持，但是无法支持Spring的许多插件，例如AOP功能、Web应用等。

该容器方法签名如下：

+ `<T> T getBean(String name, Class<T> requiredType) throws BeansException`：根据配置的name和id以及Bean类型取得配置的Bean对象，由于使用的类型参数直接取得的类对象，所以不需要强制转型；
+ `Object getBean(String name) throws BeansException`：此方法是根据name或id获取Bean对象，由于没有指定类型，所以需要进行强制类型转换；
+ `<T> T getBean(Class<T> requiredType) throws BeansException`：该方法只接受一个bean4de类型参数，会取得满足此类型的第一个bean对象。通常情况下，每个Bean类型只注册一个Bean对象，所以可以通过此种方式来取得Bean对象。

该容器接口的实现类为`XmlBeanFactory`，其构造方法语法为`XmlBeanFactory(Resource resource)`。

该构造方法接收一个`Resource`类型的对象，该对象表示一个资源文件，Resource接口的实现类如下：

+ `org.springframework.core.io.ClassPathResource`：该实现类用于读取类路径下的资源文件的资源实现类；
+ `org.springframework.core.io.FileSystemResource`：该实现类用于读取操作系统物理路径下的XML配置文件。

示例代码如下：

先是Bean接口
```
package spring.feily.tech;

public interface SpringBean {

	public abstract void sayHello();
	
}
```

再是Bean实现

```
package spring.feily.tech;

import org.springframework.stereotype.Component;

public class SpringBeanImpl implements SpringBean {

	@Override
	public void sayHello() {
		System.out.println("hello, world");
	}
}
```

配置文件如下

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context.xsd">
  <bean id="springBean" class="spring.feily.tech.SpringBeanImpl"></bean>
</beans>
```

主文件如下，测试BeanFactory取得对象的三种方法

```
package spring.feily.tech;

import org.springframework.beans.factory.BeanFactory;
import org.springframework.beans.factory.xml.XmlBeanFactory;
import org.springframework.core.io.ClassPathResource;

public class SpringBeanMain {

	@SuppressWarnings("deprecation")
	public static void main(String[] args) {
		BeanFactory beanFactory = new XmlBeanFactory(new ClassPathResource("context.xml"));
		SpringBean sb = (SpringBean) beanFactory.getBean("springBean");
		sb.sayHello();
		SpringBean sb1 = beanFactory.getBean(SpringBeanImpl.class);
		sb1.sayHello();
		SpringBean sb2 = beanFactory.getBean("springBean", SpringBeanImpl.class);
		sb2.sayHello();
	}

}
```

运行结果为

```
七月 13, 2019 3:15:48 下午 org.springframework.beans.factory.xml.XmlBeanDefinitionReader loadBeanDefinitions
信息: Loading XML bean definitions from class path resource [context.xml]
hello, world
hello, world
hello, world
```

## 三、ApplicationContext容器

`ApplicationContext`接口除了继承了`BeanFactory`接口，还继承了`ApplicationEventPublisher`、`EnvironmentCapable`、`HierarchicalBeanFactory`、`ListableBeanFactory`、`MessageSource`、`ResourceLoader`、`ResourcePatternResolver`等与Spring容器相关的环境和事件接口，实现了更高级的适合于企业级应用的Spring IoC容器。

`ApplicationContext`具备所有`BeanFactory`的方法，还包含其他父接口的方法。

`ApplicationContext`的常用实现类如下：

实现类名称 | 使用场合
:-: | :-:
`ClassPathXmlApplicationContext` | 通用
`FileSystemXmlApplicationContext` | 通用
`AnnotationConfigApplicationContext` | 通用

这里演示`ClassPathXmlApplicationContext`和`AnnotationConfigApplicationContext`实现类的使用方法，如下

`ClassPathXmlApplicationContext`基于上述代码，只修改主文件，如下

```
package spring.feily.tech;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class SpringBeanMain {

	public static void main(String[] args) {
		ApplicationContext applicationContext = new ClassPathXmlApplicationContext("context.xml");
		SpringBean sb = applicationContext.getBean("springBean", SpringBeanImpl.class);
		sb.sayHello();
	}

}
```

`AnnotationConfigApplicationContext`实现类的代码如下

首先修改`SpringBean`的实现类`SpringBeanImpl`为注解配置方式

```
package spring.feily.tech;

import org.springframework.stereotype.Component;

@Component("springBean")
public class SpringBeanImpl implements SpringBean {

	@Override
	public void sayHello() {
		System.out.println("hello, world");
	}
}
```

此处不需要xml文件，采用配置类方式，配置类为

```
package spring.feily.tech;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@Configuration
@ComponentScan(basePackages = "spring.feily.tech")
public class SpringConfigurator {

}
```

主文件为

```
package spring.feily.tech;

import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class SpringBeanMain {

	public static void main(String[] args) {
		ApplicationContext applicationContext = new AnnotationConfigApplicationContext(tech.feily.spring.SpringConfigurator.class);
		SpringBean sb = applicationContext.getBean("springBean", SpringBeanImpl.class);
		sb.sayHello();
	}

}
```

## 四、BeanFactory与ApplicationContext对比

+ BeanFactory为轻量级IoC容器，但是无法支持Spring的许多插件，ApplicationContext则相反；
+ ApplicationContext容器是一个勤奋型的容器，当创建ApplicationContext类型的IoC容器后，该容器就会自动创建所有注册的Bean的实例对象，而BeanFactory容器则是一个懒惰型的IoC容器，它在创建BeanFactory的IoC容器后，不会立即创建注册的Bean对象，而是调用getBean方法后才会创建请求的Bean的实例对象。