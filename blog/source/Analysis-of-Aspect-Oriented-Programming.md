title: "面向切面编程(Aspect Oriented Programming)浅析"
date: 2018-12-13 20:49:50 +0800
update: 2018-12-13 20:49:50 +0800
author: me
cover: "-/images/Spring-Framework.jpg"
tags:
    - Spring Framework
preview: 把通用性非功能性代码的实现方式就叫做面向切面编程，切面是贯穿于业务代码的，这就是AOP。

---

面向接口编程通过结合IoC和DI，使得耦合性相当低，可拓展性极强，实现了低耦合高内聚，但是这种编程方式是对功能性主业务需求的分而治之，对于一些非功能性的需求则显得力不从心，比如日志、安全、事务、性能统计等次要但很必要的功能就显得不够省力。

我们在软件系统设计的时候，一项很重要的工作就是把一个大系统按业务功能分解成一个个低耦合、高内聚的模块，分而治之，但是分解之后就会发现，有些非功能性需求的东西是通用的或者说是跨越多个模块的，即多个模块都需要，比如

1. 日志：对特定的操作输出日志来记录； 
2. 安全：在执行操作之前进行权限的检查； 
3. 事务：在方法开始之前要开始事务，在方法结束之后要提交或者回滚事务；
4. 性能统计：统计每个方法的执行时间
5. ......

这些多个模块都需要的非功能性需求用面向接口编程的方式实现是很不优雅的，但是用面向切面的编程方式来实现则方便很多。

主业务功能性需求我们可以看作是一个个面，每层叠加在一起构成了一个面包，这个面包就是整个需求，由于每个面都需要一些非功能性需求(诸如日志、安全等等)，那么这些需求就像一个切面一样贯穿到了主业务需求的每层面包当中，即非功能性代码与业务代码的关注点不同，业务代码的关注点是必要性功能的实现(面向接口更简单，强调的是可拓展性)，非功能性代码的关注点是通用性非必要功能的实现，采用面向切面的方式更简单，即非功能性代码与业务代码之间应该是正交的(垂直的)。

把通用性非功能性代码的实现方式就叫做面向切面编程，切面是贯穿于业务代码的。这就是AOP

先说明几个简单的概念

1. 切入点或连接点：如果切面要切入业务代码的某个方法，那么这个方法就是切面的切入点； 
2. 通知：由于切面是对业务代码通用性非必要功能的抽象，那么以日志为例，比如日志切面要切入业务需求User类的print()方法，那么就要在该方法执行之前和之后记录日志，这就要用到通知，通知就是对切入方法的前置、后置和环绕等操作；
3. 目标：即切面切入的方法所在的类就是目标，例如User类就是目标。

通过一个简单的例子来说明之

User类如下，该类通过Setter方式接受参数创建bean
```
package springTest;

public class User {
    
    String userName;
    
    public void setUserName(String userName) {
        this.userName = userName;
    }
    
    public String getUserName() {
        return userName;
    }
    
    public void print() {
        System.out.println("I am " + userName);
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
<bean id="u" class="springTest.User">
    <property name="userName" value="Java"/>	<!-- 传递参数创建bean -->
</bean>
</beans>
```
主文件如下
```
package springTest;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class UserMain {

    public static void main(String[] args) {
        ApplicationContext factory = new ClassPathXmlApplicationContext("springTest/userBean.xml");
        User user = (User) factory.getBean("u");
        user.print();
    }

}
```
运行结果为
```
I am Java
```
假如我们现在要给目标User类的print方法记录操作日志，那么就需要一个切面，定义如下
```
package springTest;

import java.util.Date;

public class Log {

    public void beforeExecute() {
        System.out.println("Log : " + new Date() + "User类的print方法即将执行");
    }
    
    public void afterExecute() {
        System.out.println("Log : " + new Date() + "User类的print方法已经执行");
    }
}
```
可以看出，其实就是一个简单的Java类，定义了两个方法，在目标方法执行之前执行beforeExecute()方法记录日志，在目标方法执行之后执行afterExecute()方法记录日志，但是并没有看到切面Log作用在目标User类之上啊？？？稍安勿躁，这些都是配置文件完成的，如下
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
<bean id="u" class="springTest.User">
    <property name="userName" value="Java"/>
</bean>
<bean id="log" class="springTest.Log"/>
<aop:config proxy-target-class="true" >
<aop:aspect ref="log">
    <aop:pointcut id="place-order" expression="execution( * springTest.User.print(..))"/>
    <aop:before pointcut-ref="place-order" method="beforeExecute"/>
    <aop:after pointcut-ref="place-order" method="afterExecute"/>
</aop:aspect>
</aop:config>
</beans>
```
我们通过`<bean id="log" class="springTest.Log"/>`创建了切面Log的bean实例，然后通过`<aop:aspect ref="log">`将切面Log的实例log作为切面，然后定义切入点pointcut为sprintTest包内User类的print方法(代号为place-order)，即切面作用于该方法，只通知该方法，然后对该切入点place-order绑定前置和后置操作，即beforeExecute方法和afterExecute方法记录User类print方法的日志

其余代码不变，然后运行，结果为
```
Log : Thu Dec 13 19:57:21 CST 2018User类的print方法即将执行
I am Java
Log : Thu Dec 13 19:57:21 CST 2018User类的print方法已经执行
```
很明显，日志记录成功。

如果我们要监控User类中的其他方法那怎么办？

我们不妨再给User类中再增加两个方法，如下
```
package springTest;

public class User {
    
    String userName;
    
    public void setUserName(String userName) {
        this.userName = userName;
    }
    
    public String getUserName() {
        return userName;
    }
    
    public void print() {
        System.out.println("I am " + userName);
    }
    
    public void print1() {
        System.out.println("I am " + userName + " 1");
    }

    public void print2() {
        System.out.println("I am " + userName + " 2");
    } 
}
```
其实我们只需要修改配置文件切面的作用域即可，如下
```
<aop:pointcut id="place-order" expression="execution( * springTest.User.*(..))"/>
```
然后再在主文件中调用一下其余两个方法就可以看到效果了
```
package springTest;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class UserMain {

    public static void main(String[] args) {
        ApplicationContext factory = new ClassPathXmlApplicationContext("springTest/userBean.xml");
        User user = (User) factory.getBean("u");
        user.print();
        user.print1();
        user.print2();
    }

}
```
结果为
```
Log : Thu Dec 13 20:00:46 CST 2018User类的print方法即将执行
I am Java
Log : Thu Dec 13 20:00:46 CST 2018User类的print方法已经执行
Log : Thu Dec 13 20:00:46 CST 2018User类的print方法即将执行
I am Java 1
Log : Thu Dec 13 20:00:46 CST 2018User类的print方法已经执行
Log : Thu Dec 13 20:00:46 CST 2018User类的print方法即将执行
I am Java 2
Log : Thu Dec 13 20:00:46 CST 2018User类的print方法已经执行
```
怎么样，简单吧？如果要给其它类也要记录日志怎么办？同样修改作用域，将类名部分用通配符*代替即可。

这是一个较为简单优雅的AOP实例，还有一个更为复杂的，不妨也看一下(这个是真的复杂)

假定我们现在有一个银行账户Account类，里面有add和minus方法，要给这两个方法加日志功能，包括前置、后置和环绕操作，那么首先要先定义一个Account接口
```
package springTest;

public interface Account {
    void add(int money);
    void minus(int money);
}
```
再实现这个接口
```
package springTest;

public class AccountImpl implements Account {

    private String name;
    
    public String getName() {
        return name;
    }
    
    public void setName(String name) {
        this.name = name;
    }
    
    @Override
    public void add(int money) {
        System.out.println("给" + name + "账户加钱" + money + "元");
    }

    @Override
    public void minus(int money) {
        System.out.println("给" + name + "账户扣钱" + money + "元");
    }

}
```
那么，我们切面的目标就是这个Account接口的实现类AccountImpl。接下来定义前置、后置和环绕类及方法，如下
```
package springTest;

import java.lang.reflect.Method;
import org.springframework.aop.MethodBeforeAdvice;


public class BeforeAdvice implements MethodBeforeAdvice {

    public void before(Method m, Object[] args, Object target) throws Throwable {
        System.out.println("在方法调用前");
        System.out.println("执行的方法是" + m);
        System.out.println("方法的参数是" + args[0]);
        System.out.println("目标对象是：" + target);
    }
}

package springTest;

import java.lang.reflect.Method;

public class AfterAdvice implements org.springframework.aop.AfterAdvice {

    public void afterReturning(Object returnValue, Method m, Object[] args, Object target) throws Throwable {
        System.out.println("在方法调用后");
        System.out.println("执行的方法是" + m);
        System.out.println("方法的参数是" + args[0]);
        System.out.println("目标对象是：" + target);
        
    }
}

package springTest;

import org.aopalliance.intercept.MethodInterceptor;
import org.aopalliance.intercept.MethodInvocation;

public class AroundInterceptor implements MethodInterceptor {

    @Override
    public Object invoke(MethodInvocation arg0) throws Throwable {
        System.out.println("调用方法之前：invocation对象：["+ arg0 + "]");
        Object rval = arg0.proceed();
        System.out.println("调用结束...");
        return rval;
    }

}
```
主文件为
```
package springTest;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class AOPMain {

    public static void main(String[] args) {
        ApplicationContext factory = new ClassPathXmlApplicationContext("springTest/AOPBean.xml");
        Account account = (Account) factory.getBean("account");
        System.out.println("第一个add方法");
        account.add(100);
        System.out.println("第二个minus方法");
        account.minus(200);
    }

}
```
同样的，通过配置文件绑定切面和目标，如下
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

<!-- 先给目标对象注入参数以创建bean -->
<bean id="accountTarget" class="springTest.AccountImpl">
    <property name="name" value="Java" />
</bean>

<!-- 再配置前置、环绕bean -->
<bean id="myBeforeAdvice" class="springTest.BeforeAdvice"/>
<bean id="myAroundInterceptor" class="springTest.AroundInterceptor"/>

<!-- 后置bean比较特殊，将20行的后置bean包装在id为addAdvisor的bean当中，并指定patterns使该后置bean只作用在add方法上，对其他方法没有作用 -->
<bean id="addAdvisor" class="org.springframework.aop.support.RegexpMethodPointcutAdvisor">
    <property name="advice">
        <bean class="springTest.AfterAdvice"/>
    </property>
    <property name="patterns">
        <list>
            <value>.*add*.</value>	<!-- 只对add方法管用 -->
        </list>
    </property>
</bean>

<!-- 以上是配置前后置及环绕bean -->
<!-- 以下是绑定切面(通知)和目标 -->

<bean id="account" class="org.springframework.aop.framework.ProxyFactoryBean">
    <property name="proxyInterfaces" value="springTest.Account"/>
    <property name="target">	<!-- 切面的目标是id为accountTarget的bean类，即上面定义的 -->
        <ref bean="accountTarget"/>
    </property>
    <property name="interceptorNames">	<!-- 拦截器列表，对目标方法拦截以记录日志 -->
        <list>
            <value>myBeforeAdvice</value>
            <value>myAroundInterceptor</value>
        </list>
    </property>
</bean>
</beans>
```
执行结果为
```
第一个add方法
在方法调用前
执行的方法是public abstract void springTest.Account.add(int)
方法的参数是100
目标对象是：springTest.AccountImpl@12c8a2c0
调用方法之前：invocation对象：[ReflectiveMethodInvocation: public abstract void springTest.Account.add(int); target is of class [springTest.AccountImpl]]
给Java账户加钱100元
调用结束...
第二个minus方法
在方法调用前
执行的方法是public abstract void springTest.Account.minus(int)
方法的参数是200
目标对象是：springTest.AccountImpl@12c8a2c0
调用方法之前：invocation对象：[ReflectiveMethodInvocation: public abstract void springTest.Account.minus(int); target is of class [springTest.AccountImpl]]
给Java账户扣钱200元
调用结束...
```
很明显，这种方法很繁琐，我们完全可以将其改造一下，如下

目标类Account，为了与上文区分(下同)，我们定义为Accounts，如下
```
package springTest;

public class Accounts {

    private String name;
    
    public String getName() {
        return name;
    }
    
    public void setName(String name) {
        this.name = name;
    }
    
    public void add(int money) {
        System.out.println("给" + name + "账户加钱" + money + "元");
    }

    public void minus(int money) {
        System.out.println("给" + name + "账户扣钱" + money + "元");
    }

}
```
切面类，我们定义为AccountsLog，如下
```
package springTest;

import java.lang.reflect.Method;

import org.aopalliance.intercept.MethodInterceptor;
import org.aopalliance.intercept.MethodInvocation;
import org.springframework.aop.MethodBeforeAdvice;

public class AccountsLog implements 
    MethodBeforeAdvice, org.springframework.aop.AfterAdvice, MethodInterceptor {

    @Override
    public Object invoke(MethodInvocation arg0) throws Throwable {
        System.out.println("调用方法之前：invocation对象：["+ arg0 + "]");
        Object rval = arg0.proceed();
        System.out.println("调用结束...");
        return rval;
    }

    @Override
    public void before(Method m, Object[] args, Object target) throws Throwable {
        System.out.println("在方法调用前");
        System.out.println("执行的方法是" + m);
        System.out.println("方法的参数是" + args[0]);
        System.out.println("目标对象是：" + target);
    }


    public void afterReturning(Object returnValue, Method m, Object[] args, Object target) throws Throwable {
        System.out.println("在方法调用后");
        System.out.println("执行的方法是" + m);
        System.out.println("方法的参数是" + args[0]);
        System.out.println("目标对象是：" + target);
        
    }
}
```
该类实现了三个接口，分别是前置，后置和环绕接口，因为我们要拿到相应的参数，所以必须实现接口并重写方法，如果不拿参数，自己写即可不需要实现接口

配置文件为
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
<bean id="acc" class="springTest.Accounts">
    <property name="userName" value="Java"/>
</bean>
<bean id="log" class="springTest.AccountsLog"/>
<aop:config proxy-target-class="true" >
<aop:aspect ref="log">
    <aop:around pointcut="execution(* springTest.Accounts.*(..))" method="invokev" />
    <aop:before pointcut="execution(* springTest.Accounts.*(..))" method="before"/>
    <aop:after pointcut="execution(* springTest.Accounts.*(..))" method="afterReturning"/>
</aop:aspect>
</aop:config>
</beans>
```
这次我们并没有像第一个例子一样单独定义切入点，而是对不同的通知类型单独定义，其实完全可以提出来，这样做是为了演示这种方式，主文件为
```
package springTest;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class AccountsMain {

    public static void main(String[] args) {
        ApplicationContext factory = new ClassPathXmlApplicationContext("springTest/accountsBean.xml");
        Account account = (Account) factory.getBean("account");
        System.out.println("第一个add方法");
        account.add(100);
        System.out.println("第二个minus方法");
        account.minus(200);
    }

}
```
运行结果为
```
第一个add方法
在方法调用前
执行的方法是public abstract void springTest.Account.add(int)
方法的参数是100
目标对象是：springTest.AccountImpl@12c8a2c0
调用方法之前：invocation对象：[ReflectiveMethodInvocation: public abstract void springTest.Account.add(int); target is of class [springTest.AccountImpl]]
给Java账户加钱100元
调用结束...
第二个minus方法
在方法调用前
执行的方法是public abstract void springTest.Account.minus(int)
方法的参数是200
目标对象是：springTest.AccountImpl@12c8a2c0
调用方法之前：invocation对象：[ReflectiveMethodInvocation: public abstract void springTest.Account.minus(int); target is of class [springTest.AccountImpl]]
给Java账户扣钱200元
调用结束...
```
怎么样？完美无缺！！！

需要特别注意的是AOP的使用，不仅仅要用Spring官方给出的jar，还需要两个jar，分别从这里下载

+ [https://www.findjar.com/jar/aopalliance/jars/aopalliance-1.0.jar.html](https://www.findjar.com/jar/aopalliance/jars/aopalliance-1.0.jar.html)
+ [http://maven.outofmemory.cn/org.aspectj/aspectjweaver/1.7.4/](http://maven.outofmemory.cn/org.aspectj/aspectjweaver/1.7.4/)
