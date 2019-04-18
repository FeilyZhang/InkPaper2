title: "Java异常及异常处理机制"
date: 2018-11-30 20:31:57 +0800
update: 2018-11-30 20:31:57 +0800
author: me
cover: "-/images/java.jpg"
tags:
    - Java Core
preview: 受检异常和未受检异常的区别在于Java对这两种异常的态度，对于受检异常Java要求程序员处理，否则会有编译错误。

---

## 一、比较常见的异常

### 1.1 NullPointerException

先看一段代码

```
public class ExceptionTest {

    public static void main(String[] args) {
	      String string = null;
	      string.indexOf('s');
	      System.out.println("end");
    }

}
```

输出为

```
Exception in thread "main" java.lang.NullPointerException
	at ExceptionTest.main(ExceptionTest.java:6)
```

空指针异常，即被擦操作对象不存在，即空指针，那么就会抛出该异常。

### 1.2 NumberFormatException

```
import java.util.Scanner;

public class ExceptionTest {

    public static void main(String[] args) {
	      System.out.println(Integer.parseInt(new Scanner(System.in).next()));
    }

}
```

输出为

```
a
Exception in thread "main" java.lang.NumberFormatException: For input string: "a"
	at java.lang.NumberFormatException.forInputString(Unknown Source)
	at java.lang.Integer.parseInt(Unknown Source)
	at java.lang.Integer.parseInt(Unknown Source)
	at ExceptionTest.main(ExceptionTest.java:6)
```

即被操作对象不是数字格式，就会抛出该异常。

### 1.3 ArrayIndexOutOfBoundsException

```
public class ExceptionTest {

    public static void main(String[] args) {
        int[] array = {0, 1, 2};
        System.out.println(array[array.length]);
    }

}
```

```
Exception in thread "main" java.lang.ArrayIndexOutOfBoundsException: 3
	at ExceptionTest.main(ExceptionTest.java:6)
```

数组下标越界，就会抛出该异常

### 1.4 StringIndexOutOfBoundsException

```
public class ExceptionTest {

    public static void main(String[] args) {
        String string = "hello , world";
        System.out.println(string.substring(string.length() + 1));
    }

}
```

输出为

```
Exception in thread "main" java.lang.StringIndexOutOfBoundsException: String index out of range: -1
	at java.lang.String.substring(Unknown Source)
	at ExceptionTest.main(ExceptionTest.java:6)
```

对String型操作数进行操作，超出其length就会触发该异常

## 二、对异常的处理

当然是使用try-catch-finally语句进行处理

可能的组合如下

* try-catch 捕获异常并处理
* try-finally 不捕获异常，即异常会抛出，但仍会执行finally中的语句，即异常并不会中断程序的执行(仅限于finally中的语句会执行)
* try-catch-finally 捕获并处理异常，然后一定会执行finally中的语句。

需要注意的是如果没有用try-catch-finally语句处理异常，那么就会在异常发生的位置中断程序的执行。 示例如下

```
public class ExceptionTest {

    public static void main(String[] args) {
        String string = "hello , world";
        try {
            System.out.println(string.substring(string.length() + 1));
        } catch(StringIndexOutOfBoundsException e) {
	          System.out.println("String型下标越界");
	      } finally {
	          System.out.println("over");
	      }
    }

}
```

输出为

```
String型下标越界
over
```

另外，catch块中为相应异常类的对象e，可以通过该类的`getMessage()`方法和`getCause()`方法获取异常的描述和原因，例如上述代码可以获取到的信息如下

```
System.out.println(e.getMessage() + ", " + e.getCause());    //String index out of range: -1, null
```

## 三、异常类与异常类体系

所有的异常类都有一个共同的父类，即Throwable，该父类有四个构造方法，如下

```
public Throwable()
public Throwable(String message)
public Throwable(Throwable cause)
public Throwable(String message, Throw cause) message为异常消息，即对该异常的描述；cause描述触发该异常的其他异常，也就是说异常可能会形成异常链，上层的异常会有由底层的异常触发。 Throwable的一些常用方法如下
void printStackTrace()    //打印异常栈信息到标准错误输出流
String getMessage()    
Throwable getCause()
```

以Throwable为根，Java定义了很多的异常类，形成了异常类体系，部分如下

```
Throwable
  |-Error
    |-VirtualMachineError
	|-OutOfMemoryError
	|-StackOverflowError
  |-Exception
    |-IOException
	|-SQLException
	|-RuntimeException
	  |-IllegalStateException
	  |-NullPointerException
	  |-ClassCastException
	  |-IllegalArgumentException
	    |-NumberFormatException
	  |-IndexOutofBoundsException
	    |-StringIndexOutOfBoundsException
	    |-ArrayIndexOutOfBoundsException
```

还可以自定义异常类，如下

```
public class AppException extends Exception {

    /**
     * 
     */
    private static final long serialVersionUID = 1L;

    public AppException() {
	      super();
    }
	
    public AppException(String message) {
	      super(message);
    }
	
    public AppException(Throwable cause) {
	      super(cause);
    }
	
    public AppException(String message, Throwable cause) {
	      super(message, cause);
    }
}
```

只需要实现ThrowAble的四个构造方法即可，那么在处理异常时，可以重新抛出已知异常通过自定义异常类再处理，这样做的话可以增加描述异常的信息或者细节，以便于更好的理解异常发生的情况或者位置，如下

```
public class ExceptionTest {

    public static void main(String[] args) {
	      String string = "hello , world";
	      try {
	          System.out.println(string.substring(string.length() + 1));
	      } catch (StringIndexOutOfBoundsException e) {
	          System.out.println(e.getMessage() + ", " + e.getCause());
	          try {
		            throw new AppException("下标越界", e);
	          } catch (AppException e1) {
		            // TODO Auto-generated catch block
		            System.out.println(e1.getMessage() + ", " + e1.getCause());
	          }
	            } finally {
	          System.out.println("over");
	      }
    }

}
```

输出为

```
String index out of range: -1, null
下标越界, java.lang.StringIndexOutOfBoundsException: String index out of range: -1
over
```

我们先处理`StringIndexOutOfBoundsException`的异常，即打印`e.getMessage()`和`e.getCause()`之后，然后通过`throw`关键字重新抛出该异常，将异常传递给`AppException`，这样就形成了一条异常链，顶层`AppException`由底层`StringIndexOutOfBoundsException`触发，那么我们通过`try-catch`语句处理该异常就可以获取相信信息，即

```
System.out.println(e1.getMessage() + ", " + e1.getCause());    //下标越界, java.lang.StringIndexOutOfBoundsException: String index out of range: -1
```

也可以通过`throw`关键字直接在方法层面抛出`AppException`异常，在处理完异常链之后才会执行`finally`中的语句。

## 四、try-with-resources

对于一些使用资源的场景，例如操作文件和数据库，典型的使用流程是先打开资源，然后操作，最后在finally语句中释放资源，Java7支持这种语法形式，即try-catch-resources.这种语法针对实现了java.lang.AutoCloseable接口的对象，该接口定义如下

```
public interface AutoCloseable {
    void close() throws Exception;
}
```

没有try-with-resources时，操作资源的形式如下

```
public static void useResource() throws Exception {
    AutoCloseable autoCloseable = new FileInputStream("hello");
    try {
        //使用资源
    } finall {
        autoCloseable.close();
    }
}
```

使用了try-with-resources语法，形式如下

```
public static void useResource() throws Exception {
    try ( AutoCloseable autoCloseable = new FileInputStream("hello")) {
        //使用资源
    }
}
```

也就是在try语句执行完毕之后，系统会自动调用close()方法释放资源。

## 五、throws关键字

throws可以抛出一系列异常，跟在方法括号的后面，含义为这个方法可能会抛出这些异常(或其中的某个)且没有对这些异常进行处理或者没处理完。

## 六、受检异常和未受检异常

RuntimeException的真实含义是未受检异常，Exception的除RuntimeException之外的其他异常则是受检异常，受检异常和未受检异常的区别在于Java对这两种异常的态度，对于受检异常Java要求程序员处理，否则会有编译错误。

关于两者的详细区别，一种普遍说法是：未受检异常说明程序本身存在逻辑错误，比如NPE，这些在编程时完全可以避免；受检异常则是说明程序本身没有错误但是涉及到I/O、网络或者文件以及数据库的操作时，可能不可避免的会出现问题，虽然程序本身不存在逻辑问题，但是也应该进行处理。

简言之，未受检异常是思维考虑问题不全面而产生的，完全可以避免；受检异常不是思维的问题，而是资源利用或者资源本身性质而引发的问题，不可避免但是可以尽可能捕获作进一步处理。
