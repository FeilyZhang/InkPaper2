title: "SpringMVC控制器使用指南"
date: 2019-07-14 11:35:49 +0800
update: 2019-07-14 11:35:49 +0800
author: me
cover: "-/images/springmvc.jpg"
tags:
    - Spring
preview: SpringMVC控制器使用全流程简单示例。

---

注意，一定要将`library`引用的jar包放到`web-inf/lib`下，前者都很熟悉，后者放置的办法是`右键项目 -> Properties -> Deployment Assembly -> Add -> Java Build Path Entries -> 全选 -> Finish -> Apply ->Apply and Close`，否则会报错`java.lang.ClassNotFoundException: org.springframework.web. servlet.DispatcherServlet`。

接下来，首先定义`web-inf/web.xml`文件，分别是定义`servlet-name`及对应的`servlet-class`，以及该`servlet-name`定义的的映射了`url`与`controller`对应关系的SpringMVC配置文件。然后通过`servlet-name`与`url-pattern`来指明所有的请求路径均由`servlet-name`指明的servlet来处理，是怎么处理的呢？就是请求先到`web.xml`文件，然后文件指明了请求由SpringMVC的配置文件来处理，于是`DispatcherServlet`读取SpringMVC的配置文件，并扫描`Controller`包，根据`RequestMapping`配置的路径来分派处理该请求的Controller。配置示例如下

```
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://java.sun.com/xml/ns/javaee" xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd" id="WebApp_ID" version="3.0">
  <display-name>SpringMVC</display-name>
  <welcome-file-list>
    <welcome-file>index.html</welcome-file>
    <welcome-file>index.htm</welcome-file>
    <welcome-file>index.jsp</welcome-file>
    <welcome-file>default.html</welcome-file>
    <welcome-file>default.htm</welcome-file>
    <welcome-file>default.jsp</welcome-file>
  </welcome-file-list>
  
  <!-- 从这里开始 -->
  <servlet>
    <servlet-name>DispatcherServlet</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>classpath:springmvc.xml</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
  </servlet>
  <servlet-mapping>
    <servlet-name>DispatcherServlet</servlet-name>
    <url-pattern>/</url-pattern>
  </servlet-mapping>
</web-app>
```

接下来必然是配置SpringMVC的配置文件，该文件位于根目录下，即`src/springmvc.xml`,该文件主要是开启SpringMVC注解的配置以及扫描路径，主要是

```
<!-- 不同于Spring的<context:annotationconfig/>标签，功能可以理解为是一样的 -->
<bean class="org.springframework.web.servlet.mvc.annotation.AnnotationMethodHandlerAdapter"/>
<context:component-scan base-package="tech.feily.springmvc.controller"/>
```

上述配置开启了SpringMVC的注解，这样`DispatcherServlet`就知道根据配置的扫描路径的包找到url对应的controller。然后还有配置视图解析的相关内容，如下

```
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
  <property name="prefix" value="/admin"/>
  <property name="suffix" value=".jsp"/>
</bean>
```

至此，简单完整的springmvc配置文件整体如下

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:mvc="http://www.springframework.org/schema/mvc"
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:tx="http://www.springframework.org/schema/tx"
    xsi:schemaLocation="
        http://www.springframework.org/schema/tx
        http://www.springframework.org/schema/tx/spring-tx-4.0.xsd
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
        http://www.springframework.org/schema/mvc
        http://www.springframework.org/schema/mvc/spring-mvc-4.0.xsd     
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context-4.0.xsd">
    <bean class="org.springframework.web.servlet.mvc.annotation.AnnotationMethodHandlerAdapter"/>
    <context:component-scan base-package="tech.feily.springmvc.controller"/>
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/admin"/>
        <property name="suffix" value=".jsp"/>
    </bean>
</beans>
```

接下来便是定义控制器了，如下

```
package tech.feily.springmvc.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
public class ControllerClass {

	@RequestMapping(value = "/hello", method = RequestMethod.GET)
	public @ResponseBody String hello() {
		return "hello, world";
	}
}
```

然后运行，测试结果，如下

```
C:\Users\Administrator
λ curl localhost:8080/SpringMVC/hello
hello, world
```

搞定！