title: "Spring依赖注入的实现方式"
date: 2019-07-13 15:59:49 +0800
update: 2019-07-13 15:59:49 +0800
author: me
cover: "-/images/Spring-Framework.jpg"
tags:
    - Spring
preview: 依赖注入简单说就是为类的属性赋值。

---

## 一、属性方式DI的实现

#### 1.1 xml方式实现DI注入

Spring在XML配置形式中，使用嵌入到`<bean>`的子标记`<property>`实现属性set方式的注入，其配置语法如下

```
<bean id="id" class="Bean全称">
  <property name="Bean需要注入的属性名" 属性名="注入的值或者对象" />
</bean>
```

其中属性名的取值如下

+ 取值为`value`：这种情况下name属性对应的值得类型为简单类型的值或String类型的值；
+ 取值为`ref`：对象类型的注入，值为配置的Bean的id；
+ 空值注入：示例为`<property name="comment"><null/></property>`;
+ 集合类型的注入：要注入数组或list集合，使用`<list>`标签，内部使用`<value>`标签对将值包含进来；要注入set集合，使用`<set>`标签，内部使用`<value>`标签对将值包含进来；要注入map集合，使用`<map>`标签，内部为`<entry key="keyName"><value>keyValue</value></entry>`；property集合使用<property>标签，内部为`<props><prop key="keyName">keyValue</prop></props>`。**值得注意的是如果注入的是对象就不能使用`<value>`标签，要使用`<rel bean="beanid"/>`标签。**

相应的Bean实体中必须针对每个属性编写set方法。

示例代码如下

要注入的实体类为

```
package spring.feily.tech;

import java.util.List;
import java.util.Set;

public class User {

	private String name;
	private int age;
	private String hobby;
	private List<User> boyFriends;
	private Set<User> girlFriends;
	
	public void setName(String name) {
		this.name = name;
	}
	
	public void setAge(int age) {
		this.age = age;
	}
	
	public void setHobby(String hobby) {
		this.hobby = hobby;
	}
	
	public void setBoyFriends(List<User> boyFriends) {
		this.boyFriends = boyFriends;
	}
	
	public void setGirlFriends(Set<User> grilFriends) {
		this.girlFriends = grilFriends;
	}
	
	public String getName() {
		return name;
	}
	
	public int getAge() {
		return age;
	}
	
	public String getHobby() {
		return hobby;
	}
	
	public List<User> getBoyFriends() {
		return boyFriends;
	}
	
	public Set<User> getGirlFriends() {
		return girlFriends;
	}
	
}
```

注入的配置文件为

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context.xsd">
  <bean id="boy1" class="spring.feily.tech.User">
    <property name="name" value="Boy1"/>
    <property name="age" value="22"/>
    <property name="hobby"><null/></property>
    <property name="boyFriends"><null/></property>
    <property name="girlFriends"><null/></property>
  </bean>
  <bean id="boy2" class="spring.feily.tech.User">
    <property name="name" value="Boy2"/>
    <property name="age" value="22"/>
    <property name="hobby"><null/></property>
    <property name="boyFriends"><null/></property>
    <property name="girlFriends"><null/></property>
  </bean>
  <bean id="girl1" class="spring.feily.tech.User">
    <property name="name" value="girl1"/>
    <property name="age" value="22"/>
    <property name="hobby"><null/></property>
    <property name="boyFriends"><null/></property>
    <property name="girlFriends"><null/></property>
  </bean>
  <bean id="girl2" class="spring.feily.tech.User">
    <property name="name" value="girl2"/>
    <property name="age" value="22"/>
    <property name="hobby"><null/></property>
    <property name="boyFriends"><null/></property>
    <property name="girlFriends"><null/></property>
  </bean>
  <bean id="user" class="spring.feily.tech.User">
    <property name="name" value="Feily Zhang"/>
    <property name="age" value="21"/>
    <property name="hobby" value="Running, coding, sing song"/>
    <property name="boyFriends">
      <list>
        <ref bean="boy1"/>
        <ref bean="boy2"/>
      </list>
    </property>
    <property name="girlFriends">
      <set>
        <ref bean="girl1"/>
        <ref bean="girl2"/>
      </set>
    </property>
  </bean>
</beans>
```

测试用的主文件为

```
package spring.feily.tech;

import java.util.List;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class SpringBeanMain {

	public static void main(String[] args) {
		ApplicationContext ac = new ClassPathXmlApplicationContext("context.xml");
		User user = ac.getBean("user", User.class);
		
		System.out.println("name : " + user.getName());
		System.out.println("age : " + user.getAge());
		System.out.println("hobby : " + user.getHobby());
		
		System.out.println("BoyFriends : ");
		List<User> userBoyFriends = user.getBoyFriends();
		for (User u : userBoyFriends) {
			System.out.println("\tname : " + u.getName());
			System.out.println("\tage : " + u.getAge());
			System.out.println("\thobby : " + u.getHobby());
			System.out.println("\tboyFriends : " + u.getBoyFriends());
			System.out.println("\tgirlFriends : " + u.getGirlFriends());
			System.out.println("\t----------");
		}
		
		System.out.println("GirlFriends : ");
		List<User> userGirlFriends = user.getBoyFriends();
		for (User u : userGirlFriends) {
			System.out.println("\tname : " + u.getName());
			System.out.println("\tage : " + u.getAge());
			System.out.println("\thobby : " + u.getHobby());
			System.out.println("\tboyFriends : " + u.getBoyFriends());
			System.out.println("\tgirlFriends : " + u.getGirlFriends());
			System.out.println("\t----------");
		}
	}

}

```

输出为

```
七月 13, 2019 4:41:52 下午 org.springframework.context.support.ClassPathXmlApplicationContext prepareRefresh
信息: Refreshing org.springframework.context.support.ClassPathXmlApplicationContext@b1a58a3: startup date [Sat Jul 13 16:41:52 CST 2019]; root of context hierarchy
七月 13, 2019 4:41:52 下午 org.springframework.beans.factory.xml.XmlBeanDefinitionReader loadBeanDefinitions
信息: Loading XML bean definitions from class path resource [context.xml]
name : Feily Zhang
age : 21
hobby : Running, coding, sing song
BoyFriends : 
	name : Boy1
	age : 22
	hobby : null
	boyFriends : null
	girlFriends : null
	----------
	name : Boy2
	age : 22
	hobby : null
	boyFriends : null
	girlFriends : null
	----------
GirlFriends : 
	name : Boy1
	age : 22
	hobby : null
	boyFriends : null
	girlFriends : null
	----------
	name : Boy2
	age : 22
	hobby : null
	boyFriends : null
	girlFriends : null
	----------
```

#### 1.2 注解方式实现DI注入

+ `@Autowired`注解：该注解依靠注册Bean的类型进行自动装配，如果再此基础之上需要根据注册Bean的id注入那么再配合`@Qualifier("beanid")`注解实现。该注解的可选参数为`(required = true)`表示进行强制注入；
+ `@Resource(name = "beanid)`注解：根据注册bean的id进行装配。

由于`@Resource`注解无法实现自动注入，必须指示其标识名，因此实际编程中使用最多的是`@Autowired`注解。

注解实现装配的方式为首先在xml配置文件中声明使用`@Autowired`注解进行注入，内容为

```
<bean class="org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor"/>
```

然后被注入的类接口与实现为

```
package spring.feily.tech;

public interface SpringBean {

	public abstract void sayHello();
	
}


package spring.feily.tech;

public class SpringBeanImpl implements SpringBean {

	@Override
	public void sayHello() {
		System.out.println("hello, world");
	}
}
```

注入的类为

```
package spring.feily.tech;

import org.springframework.beans.factory.annotation.Autowired;

public class ReSayHello {

	private SpringBean springBean = null;
	
	@Autowired(required = true)
	public void setSpringBean(SpringBean springBean) {
		this.springBean = springBean;
	}
	
	public SpringBean getSpringBean() {
		return springBean;
	}
}
```

需要在xml文件中注册(或者使用注解或配置类方式注册Bean)

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context.xsd">
  <bean class="org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor"/>
  <bean id="springBean" class="spring.feily.tech.SpringBeanImpl"></bean>
  <bean id="reSayHello" class="spring.feily.tech.ReSayHello"></bean>
</beans>
```

主文件为

```
package spring.feily.tech;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class SpringBeanMain {

	public static void main(String[] args) {
		ApplicationContext ac = new ClassPathXmlApplicationContext("context.xml");
		ReSayHello say  =ac.getBean("reSayHello", ReSayHello.class);
		say.getSpringBean().sayHello();
	}

}
```

输出为

```
七月 13, 2019 5:19:30 下午 org.springframework.context.support.ClassPathXmlApplicationContext prepareRefresh
信息: Refreshing org.springframework.context.support.ClassPathXmlApplicationContext@b1a58a3: startup date [Sat Jul 13 17:19:30 CST 2019]; root of context hierarchy
七月 13, 2019 5:19:30 下午 org.springframework.beans.factory.xml.XmlBeanDefinitionReader loadBeanDefinitions
信息: Loading XML bean definitions from class path resource [context.xml]
hello, world
```

## 二、构造方法方式依赖注入的实现

#### 2.1 构造方法依赖注入的XML方式配置

使用如下标签来实现：

```
<bean id="user" class="spring.feily.tech.User">
  <constructor-arg index="0" value="Feily Zhang"/>
  <constructor-arg index="1" value="21"/>
  <constructor-arg index="2" value="Running, coding, sing song"/>
  <constructor-arg index="3">
    <list>
      <ref bean="boy1"/>
      <ref bean="boy2"/>
    </list>
  </constructor-arg>
  <constructor-arg index="4">
    <set>
      <ref bean="girl1"/>
      <ref bean="girl2"/>
    </set>
  </constructor-arg>
</bean>
```

依旧可以注入对象、简单类型和String以及集合。

需要注意的是，使用构造方法注入，在目标类中一定要有一个默认构造方法，否则会如下报错

```
No default constructor found; nested exception is java.lang.NoSuchMethodException: spring.feily.tech.User.<init>()
```

以下示例，基于构造方法XML方式注入与属性的XML方式注入

```
package spring.feily.tech;

import java.util.List;
import java.util.Set;

public class User {

	private String name;
	private int age;
	private String hobby;
	private List<User> boyFriends;
	private Set<User> girlFriends;
	
    /*
     * 以下构造方法是为了构造方法的XML注入;
     */
	public User() {
		
	}
	
	public User(String name, int age, String hobby, List<User> boyFriends, Set<User> girlFriends) {
		this.name = name;
		this.age = age;
		this.hobby = hobby;
		this.boyFriends = boyFriends;
		this.girlFriends = girlFriends;
	}
	
    /*
     * 以下set、get方法是为了属性的xml注入或者注解注入
     */
	public void setName(String name) {
		this.name = name;
	}
	
	public void setAge(int age) {
		this.age = age;
	}
	
	public void setHobby(String hobby) {
		this.hobby = hobby;
	}
	
	public void setBoyFriends(List<User> boyFriends) {
		this.boyFriends = boyFriends;
	}
	
	public void setGirlFriends(Set<User> grilFriends) {
		this.girlFriends = grilFriends;
	}
	
	public String getName() {
		return name;
	}
	
	public int getAge() {
		return age;
	}
	
	public String getHobby() {
		return hobby;
	}
	
	public List<User> getBoyFriends() {
		return boyFriends;
	}
	
	public Set<User> getGirlFriends() {
		return girlFriends;
	}
	
}
```

xml文件为

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context.xsd">
  <!-- 以下是属性的XML方式注入-->
  <bean id="boy1" class="spring.feily.tech.User">
    <property name="name" value="Boy1"/>
    <property name="age" value="22"/>
    <property name="hobby"><null/></property>
    <property name="boyFriends"><null/></property>
    <property name="girlFriends"><null/></property>
  </bean>
  <bean id="boy2" class="spring.feily.tech.User">
    <property name="name" value="Boy2"/>
    <property name="age" value="22"/>
    <property name="hobby"><null/></property>
    <property name="boyFriends"><null/></property>
    <property name="girlFriends"><null/></property>
  </bean>
  <bean id="girl1" class="spring.feily.tech.User">
    <property name="name" value="girl1"/>
    <property name="age" value="22"/>
    <property name="hobby"><null/></property>
    <property name="boyFriends"><null/></property>
    <property name="girlFriends"><null/></property>
  </bean>
  <bean id="girl2" class="spring.feily.tech.User">
    <property name="name" value="girl2"/>
    <property name="age" value="22"/>
    <property name="hobby"><null/></property>
    <property name="boyFriends"><null/></property>
    <property name="girlFriends"><null/></property>
  </bean>
  <!-- 以下是构造方法的XML方式注入-->
  <bean id="user" class="spring.feily.tech.User">
    <constructor-arg index="0" value="Feily Zhang"/>
    <constructor-arg index="1" value="21"/>
    <constructor-arg index="2" value="Running, coding, sing song"/>
    <constructor-arg index="3">
        <list>
          <ref bean="boy1"/>
          <ref bean="boy2"/>
        </list>
    </constructor-arg>
    <constructor-arg index="4">
        <set>
          <ref bean="girl1"/>
          <ref bean="girl2"/>
        </set>
    </constructor-arg>
  </bean>
</beans>
```

主文件不变，等同于本文第一节的主文件，即

```
package spring.feily.tech;

import java.util.List;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class SpringBeanMain {

    public static void main(String[] args) {
        ApplicationContext ac = new ClassPathXmlApplicationContext("context.xml");
        User user = ac.getBean("user", User.class);
        
        System.out.println("name : " + user.getName());
        System.out.println("age : " + user.getAge());
        System.out.println("hobby : " + user.getHobby());
        
        System.out.println("BoyFriends : ");
        List<User> userBoyFriends = user.getBoyFriends();
        for (User u : userBoyFriends) {
            System.out.println("\tname : " + u.getName());
            System.out.println("\tage : " + u.getAge());
            System.out.println("\thobby : " + u.getHobby());
            System.out.println("\tboyFriends : " + u.getBoyFriends());
            System.out.println("\tgirlFriends : " + u.getGirlFriends());
            System.out.println("\t----------");
        }
        
        System.out.println("GirlFriends : ");
        List<User> userGirlFriends = user.getBoyFriends();
        for (User u : userGirlFriends) {
            System.out.println("\tname : " + u.getName());
            System.out.println("\tage : " + u.getAge());
            System.out.println("\thobby : " + u.getHobby());
            System.out.println("\tboyFriends : " + u.getBoyFriends());
            System.out.println("\tgirlFriends : " + u.getGirlFriends());
            System.out.println("\t----------");
        }
    }

}
```

输出为

```
七月 13, 2019 5:42:33 下午 org.springframework.context.support.ClassPathXmlApplicationContext prepareRefresh
信息: Refreshing org.springframework.context.support.ClassPathXmlApplicationContext@b1a58a3: startup date [Sat Jul 13 17:42:33 CST 2019]; root of context hierarchy
七月 13, 2019 5:42:33 下午 org.springframework.beans.factory.xml.XmlBeanDefinitionReader loadBeanDefinitions
信息: Loading XML bean definitions from class path resource [context.xml]
name : Feily Zhang
age : 21
hobby : Running, coding, sing song
BoyFriends : 
	name : Boy1
	age : 22
	hobby : null
	boyFriends : null
	girlFriends : null
	----------
	name : Boy2
	age : 22
	hobby : null
	boyFriends : null
	girlFriends : null
	----------
GirlFriends : 
	name : Boy1
	age : 22
	hobby : null
	boyFriends : null
	girlFriends : null
	----------
	name : Boy2
	age : 22
	hobby : null
	boyFriends : null
	girlFriends : null
	----------
```

#### 2.2 构造方法的依赖注入的注解实现

直接使用注解`@Autowired`修饰目标类即可，这种方式可以不用定义默认构造方法，如下

目标类为

```
package spring.feily.tech;

import org.springframework.beans.factory.annotation.Autowired;

public class ReSayHello {

	private SpringBean springBean = null;

	@Autowired
    public ReSayHello(SpringBean springBean) {
    	this.springBean = springBean;
    }
	
	public SpringBean getSpringBean() {
		return springBean;
	}
}
```

注入的类为

```
package spring.feily.tech;

public interface SpringBean {

	public abstract void sayHello();
	
}

package spring.feily.tech;

public class SpringBeanImpl implements SpringBean {

	@Override
	public void sayHello() {
		System.out.println("hello, world");
	}
}
```

xml文件为

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context.xsd">
  <bean class="org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor"/>
  <bean id="springBean" class="spring.feily.tech.SpringBeanImpl"></bean>
  <bean id="reSayHello" class="spring.feily.tech.ReSayHello"></bean>
</beans>
```

主文件为
```
package spring.feily.tech;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class SpringBeanMain {

    public static void main(String[] args) {
        ApplicationContext ac = new ClassPathXmlApplicationContext("context.xml");
        ReSayHello say  =ac.getBean("reSayHello", ReSayHello.class);
        say.getSpringBean().sayHello();
    }

}
```

输出为

```
七月 13, 2019 5:55:04 下午 org.springframework.context.support.ClassPathXmlApplicationContext prepareRefresh
信息: Refreshing org.springframework.context.support.ClassPathXmlApplicationContext@b1a58a3: startup date [Sat Jul 13 17:55:04 CST 2019]; root of context hierarchy
七月 13, 2019 5:55:04 下午 org.springframework.beans.factory.xml.XmlBeanDefinitionReader loadBeanDefinitions
信息: Loading XML bean definitions from class path resource [context.xml]
hello, world
```