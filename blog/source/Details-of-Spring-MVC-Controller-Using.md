title: "Spring MVC控制器使用详情"
date: 2019-07-14 12:33:49 +0800
update: 2019-07-14 12:33:49 +0800
author: me
cover: "-/images/springmvc.jpg"
tags:
    - Spring
preview: Spring MVC控制器使用详情，包括控制器请求地址映射类、控制器方法参数、控制器方法返回类型。

---

## 一、控制器请求地址映射类

Spring MVC中通过`@Controller`注解来说明当前类是一个控制器类，在此基础之上通过`@RequestMapping`注解来说明请求路径。

Spring MVC`@RequestMapping(org.springframework.web.bind.annotation.RequestMapping`注释类实现控制器方法请求地址、请求方式、请求数据类型和响应数据类型等信息的映射，是控制器编程最常见的注释类，语法为

```
@RequestMapping(属性 = '值', 属性 = '值', ...)
```

该注释类的常用属性名称、取值和功能如下所述：

+ `value = String[]`：指明控制器方法请求地址，类型是`String[]`，因此可以指定多个地址(`@RequestMapping(value = {"/tomain", "/tohome"})`),如果只映射一个地址时可以不用`{}`符号，如果没有其它属性，那么可以忽略属性名直接写路径(`@RequestMapping("tomain")`)；
+ `path = String[]`：与`value`含义相同，语法相同，用于确定控制器方法的请求地址；
+ `method = RequestMethod[]`：用于确定请求方式，可以指定多个方式(`@RequestMapping(value = "/add", method = {RequestMethod.POST, RequestMethod.GET)`,也可以是一个地址(`@RequestMapping(value = "/hello", method = RequestMethod.GET)`);
+ `params = String[]`：用于指定URI中附带的参数名，示例为`@RequestMapping(value = "/add", method = RequestMethod.GET, params = {"id", "name", "pswd"})`;
+ `consumes = String[]`：用于确定请求的数据类型，可以是一种(`@RequestMapping(value = "/add", method = RequestMethod.GET, consumes = "text/plain")`或者(`@RequestMapping(value = "/add", method = RequestMethod.GET, consumes = "application/json")`))，也可以是多种(`@RequestMapping(value = "/add", method = RequestMethod.GET, consumes = {"text/plain", "application/*"})`);
+ `produces = String[]`：用于确定返回的数据类型，可以是一种(`@RequestMapping(value = "/add", method = RequestMethod.GET, produces = "text/plain")`或者(`@RequestMapping(value = "/add", method = RequestMethod.GET, consumes = "application/json; charset = UTF-8")`))也可以是多种(`@RequestMapping(value = "/add", method = RequestMethod.GET, produces = {"text/plain", "application/*"})`);
+ `header = String[]`：用于指定请求头信息，当请求满足时才受理，例如`@RequestMapping(value = "/add", method = RequestMethod.GET, header = "content-type=text/*")`。

## 二、控制器方法的常用参数

+ 使用`@RequestParam`注释的请求参数：以细粒度方式提取请求路径中的查询字符串，语法格式为`@RequestParam(属性 = "值", 属性 = "值")`,可取的属性如下
  1. `name = "参数名"‘：指定参数的名称，如果省略，则参数变量名就是参数名；
  2. `required = true | false`：指定参数是否是必需的，如果为`true`，则请求中必须有此参数，否则方法会抛出异常，默认为`true`；
  3. `defauValue = "值"`：如果参数不是必需的，当没有此参数时，控制方法中定义接收参数的值就是此属性设置的值，一般与`required = false`搭配使用；
  4. `value = "参数名"`：该属性时`name`属性的别名，与`name`属性一样用于指定参数名，不是用来指定参数的值。
+ `@PathVariable`注释的参数：用于提取REST API风格的URL地址中的参数，使用方式为先`@RequestMapping(value = "department/add/code/{code}/name/{name}")`然后在方法参数中`@PathVariable(value = "code") String code, @PathVariable(value = "name") String name)`;
+ `@RequestHeader`注释的参数：用于提取请求头的值，使用方式为在控制器方法参数中`@RequestHeader("Accept-Encoding") String encoding, @RequestHeader("Keep-Alive") long keepLive`；
+ `@RequestBody`注释的参数：用于接收前段Post的JSON数据，直接装配到Model实体中，用法为`@RequestBody Model m`;
+ `@RequestPart`注释的参数：用于取得请求中包含的文件，用于文件上传编程；
+ `CookieValue`注释的参数：用于提取Cookie值，用法为`@CookieValue(value = "userid", required = true) String userid`；
+ `javax.servlet.ServletRequest`：用于使用原始的JavaEE的请求对象；
+ `javax.servlet.http.HttpServletRequest`：用于使用HTTP请求的请求对象，可以实现一些Spring MVC无法完成的任务，例如获取客户端的IP地址；
+ `javax.servlet.ServletResponse`：用于取得原始的Java Web响应对象，使用比较少；
+ `javax.servlet.http.HttpServletResponse`：可实现专门的响应处理编程。由于Spring MVC控制器可以生成所有类型的相应处理，所以此类型参数使用较少；
+ `javax.servlet.http.HttpSession`：可用于保存会话信息，如用户的登录信息等；
+ 其他...

## 三、控制器方法的常用返回类型

+ String：返回字符串对象，通常表示View的逻辑名称，例如`return "department/add";`，地址的后缀会通过Spring MVC的配置文件补充；
+ 业务Model类：与`@ResponseBody`注解配合，直接`return model`即可；
+ Void：无返回类型；

特别注意`@ResponseBody`注解，当在此基础上返回类型是String，那么就是返回一段字符串，当是Model时，返回的就是JSON数据，也可以是List容器，那么就是JSON数组。