title: "Spring AOP的XML实现方式"
date: 2019-07-13 18:55:49 +0800
update: 2019-07-13 18:55:49 +0800
author: me
cover: "-/images/Spring-Framework.jpg"
tags:
    - Spring
preview: Spring AOP的XML实现方式代码示例。

---

被织入的类为

```
package spring.feily.tech;

public class SayHello {

	public void sayHello() {
		System.out.println("Hello, world!");
	}
}
```

前置、方法返回前、环绕实现类分别为

```
package spring.feily.tech;

import java.lang.reflect.Method;
import java.util.Date;

import org.springframework.aop.MethodBeforeAdvice;

public class BeforeAdvice implements MethodBeforeAdvice {

	@Override
	public void before(Method arg0, Object[] arg1, Object arg2) throws Throwable {
		System.out.println("Hello, current time isb " + new Date());
	}

}


package spring.feily.tech;

import java.lang.reflect.Method;
import java.util.Date;

public class AfterReturningAdvice implements org.springframework.aop.AfterReturningAdvice {

	@Override
	public void afterReturning(Object arg0, Method arg1, Object[] arg2, Object arg3) throws Throwable {
		System.out.println("Hello, current time is " + new Date());
	}

}

package spring.feily.tech;

import java.util.Date;

import org.aopalliance.intercept.MethodInvocation;

public class MethodInterceptor implements org.aopalliance.intercept.MethodInterceptor {

	@Override
	public Object invoke(MethodInvocation arg0) throws Throwable {
		Date startTime = new Date();
		Object result = arg0.proceed();
		Date endTime = new Date();
		Long runtime = endTime.getTime() - startTime.getTime();
		System.out.println("Method ： " + arg0.getMethod() + " running time is " + runtime + "ms.");
		return result;
	}

}
```

xml配置文件为

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:util="http://www.springframework.org/schema/util" 
    xmlns:aop="http://www.springframework.org/schema/aop"             
    xsi:schemaLocation="                                               
            http://www.springframework.org/schema/beans 
            http://www.springframework.org/schema/beans/spring-beans.xsd    
            http://www.springframework.org/schema/context     
            http://www.springframework.org/schema/context/spring-context.xsd    
            http://www.springframework.org/schema/mvc    
            http://www.springframework.org/schema/mvc/spring-mvc.xsd  
            http://www.springframework.org/schema/tx   
            http://www.springframework.org/schema/tx/spring-tx.xsd  
            http://www.springframework.org/schema/aop  
            http://www.springframework.org/schema/aop/spring-aop.xsd">
            
  <bean id="sayHello" class="spring.feily.tech.SayHello"/>
  
  <bean id="beforeAdvice" class="spring.feily.tech.BeforeAdvice"/>
  <bean id="afterReturnningAdvice" class="spring.feily.tech.AfterReturningAdvice"/>
  <bean id="methodInterceptor" class="spring.feily.tech.MethodInterceptor"/>
  
  <aop:config>
    <aop:pointcut id="pointCut" expression="execution(* spring.feily.tech.SayHello.*(..))" />
    <aop:advisor advice-ref="beforeAdvice" pointcut-ref="pointCut"/>
    <aop:advisor advice-ref="afterReturnningAdvice" pointcut-ref="pointCut"/>
    <aop:advisor advice-ref="methodInterceptor" pointcut-ref="pointCut"/>
  </aop:config>
</beans>
```

测试主文件为

```
package spring.feily.tech;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class AOPMain {

	public static void main(String[] args) {
		ApplicationContext ac = new ClassPathXmlApplicationContext("context.xml");
		SayHello sayHello = ac.getBean("sayHello", SayHello.class);
		sayHello.sayHello();

	}

}
```

输出为

```
七月 13, 2019 6:58:25 下午 org.springframework.context.support.ClassPathXmlApplicationContext prepareRefresh
信息: Refreshing org.springframework.context.support.ClassPathXmlApplicationContext@3d71d552: startup date [Sat Jul 13 18:58:25 CST 2019]; root of context hierarchy
七月 13, 2019 6:58:25 下午 org.springframework.beans.factory.xml.XmlBeanDefinitionReader loadBeanDefinitions
信息: Loading XML bean definitions from class path resource [context.xml]
Hello, current time isb Sat Jul 13 18:58:26 CST 2019
Hello, world!
Method ： public void spring.feily.tech.SayHello.sayHello() running time is 13ms.
Hello, current time is Sat Jul 13 18:58:26 CST 2019
```