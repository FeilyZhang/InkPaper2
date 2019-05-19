title: "Spring中Bean的配置方式"
date: 2019-05-19 16:01:14 +0800
update: 2019-05-19 16:01:14 +0800
author: me
cover: "-/images/Spring-Framework.jpg"
tags:
    - Spring
preview: Spring中的Bean配置方式，包含类-xml配置、注解-xml配置、注解-配置类配置。

---

## 一、类-xml方式

这种方式是先编写普通的Java类，然后在Spring配置文件中通过`<bean>`标签来定义bean，最后就可以通过IoC容器取得Bean实例，如下

```
package spring.feily.tech;

public interface SpringBean {

	public abstract void sayHello();
	
}
```

```
package spring.feily.tech;

public class SpringBeanImpl implements SpringBean {

	@Override
	public void sayHello() {
		System.out.println("hello, world");
	}
	
}
```

```
package spring.feily.tech;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class SpringBeanMain {

	public static void main(String[] args) {
		ApplicationContext ac = new ClassPathXmlApplicationContext("context.xml");
		SpringBean sb = ac.getBean("springBean", SpringBean.class);
		sb.sayHello();
	}

}
```

配置文件为(一定要放在src目录下)

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context-3.0.xsd">
  <!-- 
  <context:component-scan base-package="spring.feily.tech"/>
  <context:annotationconfig/>
   -->
  <bean id="springBean" class="spring.feily.tech.SpringBeanImpl"></bean>
</beans>
```

运行结果为

```
五月 19, 2019 4:00:35 下午 org.springframework.context.support.ClassPathXmlApplicationContext prepareRefresh
信息: Refreshing org.springframework.context.support.ClassPathXmlApplicationContext@b1a58a3: startup date [Sun May 19 16:00:35 CST 2019]; root of context hierarchy
五月 19, 2019 4:00:35 下午 org.springframework.beans.factory.xml.XmlBeanDefinitionReader loadBeanDefinitions
信息: Loading XML bean definitions from class path resource [context.xml]
hello, world
```

## 二、注解-xml方式

这种办法是给类加上注解，只是配置文件不需要`<bean>`标签，只需要打开注解和配置扫描路径就好了，IoC取得bean对象的代码不变，如下

```
package spring.feily.tech;

public interface SpringBean {

	public abstract void sayHello();
	
}
```

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
```
package spring.feily.tech;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class SpringBeanMain {

	public static void main(String[] args) {
		ApplicationContext ac = new ClassPathXmlApplicationContext("context.xml");
		SpringBean sb = ac.getBean("springBean", SpringBean.class);
		sb.sayHello();
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
    xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context.xsd">
  <context:component-scan base-package="spring.feily.tech"/>
  <context:annotation-config/>
</beans>
```

运行结果为

```
五月 19, 2019 4:23:25 下午 org.springframework.context.support.ClassPathXmlApplicationContext prepareRefresh
信息: Refreshing org.springframework.context.support.ClassPathXmlApplicationContext@b1a58a3: startup date [Sun May 19 16:23:25 CST 2019]; root of context hierarchy
五月 19, 2019 4:23:25 下午 org.springframework.beans.factory.xml.XmlBeanDefinitionReader loadBeanDefinitions
信息: Loading XML bean definitions from class path resource [context.xml]
hello, world
```

### 三、注解-配置类方式

这种办法是用一个被`@Configurator`和`ComponentScan(basePackages = "")`修饰的配置类来替代前两种办法中的xml配置文件的作用，IoC取得bean对象的方式也相应地变成了`AnnotationConfigApplicationContext`,如下

```
package spring.feily.tech;

public interface SpringBean {

	public abstract void sayHello();
	
}
```

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

```
package spring.feily.tech;

import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class SpringBeanMain {

	public static void main(String[] args) {
		ApplicationContext ac = new AnnotationConfigApplicationContext(SpringConfigurator.class);
		SpringBean sb = ac.getBean("springBean", SpringBean.class);
		sb.sayHello();
	}

}
```

配置类为

```
package spring.feily.tech;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@Configuration
@ComponentScan(basePackages = "spring.feily.tech")
public class SpringConfigurator {

}
```

运行结果为

```
五月 19, 2019 4:30:15 下午 org.springframework.context.annotation.AnnotationConfigApplicationContext prepareRefresh
信息: Refreshing org.springframework.context.annotation.AnnotationConfigApplicationContext@5fa7e7ff: startup date [Sun May 19 16:30:15 CST 2019]; root of context hierarchy
hello, world
```