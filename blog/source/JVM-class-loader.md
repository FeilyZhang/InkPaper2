title: "JVM类加载器"
date: 2019-04-19 19:00:25 +0800
update: 2019-04-19 19:00:25 +0800
author: me
cover: "-/images/JVM-class-loader.jpg"
tags:
    - Java
preview: 类加载器的职责就是负责类的加载，对于任意一个class，都需要由加载它的类加载器和这个类本身确立其在JVM中的唯一性，这也就是运行时包。

---

类加载器的职责就是负责类的加载，对于任意一个class，都需要由加载它的类加载器和这个类本身确立其在JVM中的唯一性，这也就是运行时包。

## 一、JVM内置的三大类加载器

JVM提供内置的三大类加载器，不同的类加载器负责将不同的类加载到JVM内存之中，并且它们之间严格遵守着父委托机制，如下图所示

![](/images/article/jvm-class-loader1.jpg)

> 图片来源：http://www.cnblogs.com/huizhi/p/10177126.html

### 1.1 根类加载器

又称Bootstrap类加载器，该类加载器是最顶层的类加载器，没有任何父加载器，由C++编写，主要负责虚拟机核心类库的加载，比如整个java.lang包都是由根加载器所加载，可以通过`-Xbootclasspath`来指定根加载器的路径,也可以通过系统属性来得知当前JVM的根加载器都加载了哪些资源。

```
public class BootstrapClassLoader {

    public static void main(String[] args) {
        System.out.println("Bootstrap:" + String.class.getClassLoader());
        System.out.println(System.getProperty("sun.boot.class.path").replace(";", "\n"));
    }

}
```

上述代码获取了String类的类加载器以及根加载器所在的加载路径，输出为

```
Bootstrap:null
C:\Program Files\Java\jdk1.8.0_201\jre\lib\resources.jar
C:\Program Files\Java\jdk1.8.0_201\jre\lib\rt.jar
C:\Program Files\Java\jdk1.8.0_201\jre\lib\sunrsasign.jar
C:\Program Files\Java\jdk1.8.0_201\jre\lib\jsse.jar
C:\Program Files\Java\jdk1.8.0_201\jre\lib\jce.jar
C:\Program Files\Java\jdk1.8.0_201\jre\lib\charsets.jar
C:\Program Files\Java\jdk1.8.0_201\jre\lib\jfr.jar
C:\Program Files\Java\jdk1.8.0_201\jre\classes
```

由于`String.class`的类加载器是根加载器，而根加载器是获取不到引用的，所以输出为null。

### 1.2 扩展类加载器

扩展类加载器的父加载器是根加载器，主要是用于加载`JAVA_HOME`下的`jre/lib/ext`子目录里面的类库，由纯Java语言实现，扩展类加载器所加载的类库可以通过系统属性`java.ext.dirs`获得，如下

```
public class ExtClassLoader {

    public static void main(String[] args) {
        System.out.println(System.getProperty("java.ext.dirs").replace(";", "\n"));
    }

}
```

输出为

```
C:\Program Files\Java\jdk1.8.0_201\jre\lib\ext
C:\Windows\Sun\Java\lib\ext
```

也可以将自己的类打包为jar，然后放到扩展类加载器所在的路径中，扩展类加载器会负责加载自己所需要的类。

### 1.3 系统类加载器

系统类加载器是一种常见的类加载器，其负责加载classpath下的类库资源。系统类加载器的父加载器是扩展类加载器，同时也是自定义类加载器的默认父加载器，系统类加载器的加载路径一般通过`-classpath`或者`-cp`指定，同样也可以通过系统属性`java.class.path`获取，如下

```
public class SysClassLoader {

    public static void main(String[] args) {
        System.out.println(System.getProperty("java.class.path").replace(";", "\n"));
        System.out.println(SysClassLoader.class.getClassLoader());
    }

}
```

输出为

```
D:\R-Java\REngine.jar
D:\R-Java\RserveEngine.jar
C:\Users\Administrator\eclipse-workspace\Demo\bin
sun.misc.Launcher$AppClassLoader@6d06d69c
```

## 二、自定义类加载器

自定义类加载器都是ClassLoader的直接子类或者间接子类，`java.lang.ClassLoader`是一个抽象类，但是里面并没有抽象方法，但是有`findClass`方法，自定义类加载器务必重写该方法，否则会抛出Class找不到的异常。

一个自定义类加载器如下所示

```
import java.io.ByteArrayOutputStream;
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;

public class MyClassLoader extends ClassLoader {

    private final static Path DEFAULT_CLASS_DIR = Paths.get("D:", "classloader");
    private final Path classDir;
    
    public MyClassLoader() {
        super();
        this.classDir = DEFAULT_CLASS_DIR;
    }
    
    public MyClassLoader(String classDir) {
        super();
        this.classDir = Paths.get(classDir);
    }
    
    public MyClassLoader(String classDir, ClassLoader parent) {
        super(parent);
        this.classDir = Paths.get(classDir);
    }
    
    
    /*
     * Method of overwriting parent class.
     * (non-Javadoc)
     * @see java.lang.ClassLoader#findClass(java.lang.String)
     */
    @Override
    protected Class<?> findClass(String name) throws ClassNotFoundException {
        // Read class binary data.
        byte[] classBytes = this.readClassBytes(name);
        // If the data is null, or no information was read, then throw ClassNotFoundException.
        if (null == classBytes || classBytes.length == 0) {
            throw new ClassNotFoundException("Can not load the class " + name);
        }
        // Calling defineClass method define class.
        return this.defineClass(name, classBytes, 0, classBytes.length);
    }
    
    /*
     * Read the class file into memory.
     */
    private byte[] readClassBytes(String name) throws ClassNotFoundException {
        // Converting a package delimiter to a file path delimiter.
        String classPath = name.replace(".", "/");
        Path classFullPath = classDir.resolve(Paths.get(classPath + ".class"));
        if (!classFullPath.toFile().exists()) {
            throw new ClassNotFoundException("The class " + name + " not found.");
        }
        try (ByteArrayOutputStream baos = new ByteArrayOutputStream()) {
            Files.copy(classFullPath, baos);
            return baos.toByteArray();
        } catch (IOException e) {
            throw new ClassNotFoundException("load the class " + name + " occur error.", e);
        }
    }
    
    @Override
    public String toString() {
        return "My ClassLoader";
    }
    
}
```

以上是一个简单的基于磁盘的类加载器，共有三个构造器，分别为

+ 第一个构造器使用默认的文件路径；
+ 第二个构造器允许从外部指定一个特定的磁盘目录；
+ 第三个构造器除了可以指定磁盘目录之外还可以指定该类加载器的父加载器。

在上述定义的自定义类加载器中，将类的权限定名转换成文件的全路径然后读取class文件的字节流数据，最后使用`defineClass`方法对class完成了定义。

以下再编写一个简单的程序，通过如下自定义类加载器进行加载。

```
public class HelloWorld {

    static {
        System.out.println("Hello World Class is Initialized.");
    }
    
    public String welcome() {
       return "Hello, World";
    }
    
}
```

如果用的是IDE，那么编译完成后需要将包下的`HelloWorld.class`文件(连同包)复制到加载器默认文件路径下并且删除IDE工作空间下的`HelloWorld.class`和`HelloWorld.java`文件，否则JVM仍然会通过系统类加载器所加载(这是由于类加载器的双亲委托机制所导致的)，接下来进行加载，如下

```
public class MyClassLoaderTest {

    public static void main(String[] args) throws ClassNotFoundException {
        MyClassLoader myClassLoader = new MyClassLoader();
        Class<?> aClass = myClassLoader.loadClass("tech.feily.doc.thread.HelloWorld");
        System.out.println(aClass.getClassLoader());
    }

}
```

输出为

```
My ClassLoader
```

可见是通过自定义类加载器加载的，这里通过一种变相的方式打破了双亲委托机制。还有两种方式来打破，等会介绍。

而且上述程序也验证了**类加载器也不会导致类的初始化**，因为HelloWorld程序中静态代码块并未得到执行，即在初始化阶段静态代码会得到执行的，但是并没有执行，所以得到了类加载器不会导致类主动初始化的结论。

如果再通过反射操作来执行代码，那么将会输出，修改一下

```
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;

public class MyClassLoaderTest {

    public static void main(String[] args) 
            throws ClassNotFoundException, InstantiationException, 
            IllegalAccessException, NoSuchMethodException, SecurityException, 
            IllegalArgumentException, InvocationTargetException {
        MyClassLoader myClassLoader = new MyClassLoader();
        Class<?> aClass = myClassLoader.loadClass("tech.feily.doc.thread.HelloWorld");
        System.out.println(aClass.getClassLoader());
        
        // Here the static code block is executed.
        Object helloWorld = aClass.newInstance();
        System.out.println(helloWorld);
        Method welcomeMethod = aClass.getMethod("welcome");
        String result = (String) welcomeMethod.invoke(helloWorld);
        System.out.println("Result: " + result);
    }

}
```

输出为

```
My ClassLoader
Hello World Class is Initialized.
tech.feily.doc.thread.HelloWorld@2328c243
Result: Hello, World
```

## 三、双亲委托机制

有一张图很好的说明了类加载器的双亲委托机制，如下

![](/images/JVM-class-loader.jpg)

> 图片来源：该图尚无法找到原始出处


上图双亲委托机制的源码描述片段为

```
public Class<?> loadClass(String name) throws ClassNotFoundException {
    return loadClass(name, false);
}

protected Class<?> loadClass(String name, boolean resolve) throws ClassNotFoundException {
    synchronized (getClassLoadingLock(name)) {
	    Class<?> c = findLoadedClass(name);    // ①
		if (c == null) {
		    long t0 = System.nanoTime();
			try {
			    if (parent != null) {	// ②
				    c = parent.loadClass(name, false);    // ③
				} else {
				    c = findBootstrapClassOrNull(name);    // ④
				}
			} catch (ClassNotFoundException e) {
			
			}
		    if (c == null) {
			    long t1 = System.nanoTime();
				c = findClass(name);    // ⑤
				sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
				sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
				sun.misc.PerfCounter.getFindClasses().increment();
			}
		}
		if (resolve) {
		    resolveClass(c);
		}
		return c;
	}
}
```

解释如下

+ ①：从当前类加载器的缓存中根据类的全路径名查询是否存在该类，如果存在则直接返回(即后面的if条件就不满足了，直接就return了)；
+ ②③：如果当前类加载器缓存中不存在该类，那么进入if判断，如果当前类存在父类加载器，那么就调用父类加载器的loadClass方法对该类进行加载，即执行③；
+ ④：如果当前类不存在父类加载器那么就用根类加载器对该类进行加载；
+ ⑤：如果当前类的所有父类加载器都没有成功加载class，那么就尝试调用当前类加载器的`findClass`方法对其进行加载，**该方法就是我们自定义类加载器需要重写的方法；**
+ 最后如果类被成功加载，就做一些性能数据的统计；
+ 由于`loadClass`指定了`resolve`为`false`，所以不会进行连接阶段的继续进行，**这也就解释了为什么通过类加载器加载类并不会导致类的初始化。**

另外两种自定义类加载器绕过双亲委托机制加载类的办法分别是：

+ 一种办法是，**直接将拓展类加载器作为自定义类加载器的父加载器，**示例代码如下

```
public class MyClassLoaderTest {

    public static void main(String[] args) throws ClassNotFoundException {
        ClassLoader extClassLoader = MyClassLoader.class.getClassLoader().getParent();
        MyClassLoader myClassLoader = new MyClassLoader("D:\\classloader", extClassLoader);
        Class<?> aClass = myClassLoader.loadClass("tech.feily.doc.thread.HelloWorld");
        System.out.println(aClass.getClassLoader());	// My ClassLoader
    }

}
```

+ 另一种办法是，**在构造自定义类加载器的时候指定父类加载器为null，**示例如下

```
public class MyClassLoaderTest {

    public static void main(String[] args) throws ClassNotFoundException {
        MyClassLoader myClassLoader = new MyClassLoader("D:\\classloader", null);
        Class<?> aClass = myClassLoader.loadClass("tech.feily.doc.thread.HelloWorld");
        System.out.println(aClass.getClassLoader());	// My ClassLoader
    }

}
```

上述破坏类加载器的双亲委托机制是通过绕过系统类加载器（SystemClassLoader）实现的，但是并没有避免一层一层的委托，还有一种更为稳健的办法就是**通过在自定义类加载器中重写`loadClass`方法来实现。**

## 四、重写loadClass方法来打破双亲委托机制

对上述自定义类加载器做如下改动，即增加重写的classLoad方法，为

```
import java.io.ByteArrayOutputStream;
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;

public class MyClassLoader extends ClassLoader {

    private final static Path DEFAULT_CLASS_DIR = Paths.get("D:", "classloader");
    private final Path classDir;
    
    public MyClassLoader() {
        super();
        this.classDir = DEFAULT_CLASS_DIR;
    }
    
    public MyClassLoader(String classDir) {
        super();
        this.classDir = Paths.get(classDir);
    }
    
    public MyClassLoader(String classDir, ClassLoader parent) {
        super(parent);
        this.classDir = Paths.get(classDir);
    }
    
    
    /*
     * Method of overwriting parent class.
     * (non-Javadoc)
     * @see java.lang.ClassLoader#findClass(java.lang.String)
     */
    @Override
    protected Class<?> findClass(String name) throws ClassNotFoundException {
        // Read class binary data.
        byte[] classBytes = this.readClassBytes(name);
        // If the data is null, or no information was read, then throw ClassNotFoundException.
        if (null == classBytes || classBytes.length == 0) {
            throw new ClassNotFoundException("Can not load the class " + name);
        }
        // Calling defineClass method define class.
        return this.defineClass(name, classBytes, 0, classBytes.length);
    }
    
    /*
     * Read the class file into memory.
     */
    private byte[] readClassBytes(String name) throws ClassNotFoundException {
        // Converting a package delimiter to a file path delimiter.
        String classPath = name.replace(".", "/");
        Path classFullPath = classDir.resolve(Paths.get(classPath + ".class"));
        if (!classFullPath.toFile().exists()) {
            throw new ClassNotFoundException("The class " + name + " not found.");
        }
        try (ByteArrayOutputStream baos = new ByteArrayOutputStream()) {
            Files.copy(classFullPath, baos);
            return baos.toByteArray();
        } catch (IOException e) {
            throw new ClassNotFoundException("load the class " + name + " occur error.", e);
        }
    }
    
    @Override
    public String toString() {
        return "My ClassLoader";
    }
    
    @Override
    protected Class<?> loadClass(String name, boolean resolve) throws ClassNotFoundException {
        synchronized(getClassLoadingLock(name)) {    // ①
		    // ②
            Class<?> cls = findLoadedClass(name);
			// ③
            if (cls == null) {
			    // ④
                if (name.startsWith("java.") || name.startsWith("javax")) {
                    try {
                        cls = getSystemClassLoader().loadClass(name);
                    } catch (Exception e) {
                        
                    }
                } else {
				    // ⑤
                    try {
                        cls = this.findClass(name);
                    } catch (ClassNotFoundException e) {
                        
                    }
					// ⑥
                    if (cls == null) {
                        if (getParent() != null) {
                            cls = getParent().loadClass(name);
                        } else {
                            cls= getSystemClassLoader().loadClass(name);
                        }
                    }
                }
            }
			// ⑦
            if (null == cls) {
                throw new ClassNotFoundException("The class " + name + " not found.");
            }
            if (resolve) {
                resolveClass(cls);
            }
            return cls;
        }
    }
    
}
```

代码还是很好理解的，与**双亲委托机制的源码描述片段**的理解同理，详细的代码解释如下

+ ①：根据类的全路径名称进行加锁，确保每一个类在多线程的情况下只被加载一次；
+ ②：到已加载类的缓存中查看该类是否已经被加载，如果已加载则直接返回；
+ ③④：若缓存中没有被加载的类，则需要对其进行首次加载，如果类的全路径以`java`和`javax`开头，(那么说明是核心类库，)则直接委托给系统类加载器对其进行加载；
+ ⑤如果类不是以`java`或`javax`开头，则尝试用我们自定义的类加载器进行加载；
+ ⑥若自定义类加载器仍旧没有完成对类的加载，则委托给其父类加载器进行加载或者系统类加载器进行加载；
+ ⑦经过若干次尝试之后，如果还是无法对类进行加载，则抛出无法找到类的异常。

然后现在即使是IDE下,不用删除`HelloWorld.java`与`HelloWorld.class`文件仍然能够通过自定义类加载器加载HelloWorld。