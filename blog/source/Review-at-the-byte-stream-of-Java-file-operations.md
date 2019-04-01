title: "再看Java文件操作之字节流"
date: 2018-12-18 11:16:30 +0800
update: 2018-12-18 11:16:30 +0800
author: me
cover: "-/images/java.jpg"
tags:
    - Java Core
preview: 这里Java以二进制字节的方式处理文件。

---

这里Java以二进制字节的方式处理文件，以二进制方式读写的主要流如下

+ InputStream/OutputStream：这是基类，都是抽象类；
+ FileInputStream/FileOutputStream：输出源和输出目标是文件的流；
+ ByteArrayInputStream/ByteArrayOutputStream：输入源和输出目标是字节数组的流
+ DataInputStream/DataOutputStream：装饰类，按基本类型和字符串而非只是字节读写流；
+ BufferedInputStream/BufferedOutputStream：装饰类，对输入输出提供缓冲功能。

## InputStream/OutputStream

InputStream的三个基本方法分别是

```
public abstract int read() throws IOException; //从流中读取下一个字节，返回类型为int，但取值为0-255，当读到流末尾时，返回-1，如果流中没有数据则会阻塞直到数据到来、流关闭或者异常出现
public int read(byte b[]) throws IOException; //批量读取，一次性读取数组b的长度个字节，返回值为实际读取的字节个数，若刚开始读取时已经到达流结尾，那么返回-1，该方法有默认实现
public int read(byte b[], int off, int len) throws IOException; //批量读取的一个重载方法
```

OutputStream的三个基本方法分别是

```
public abstract void write(int b) throws IOException; //向流中写一个字节，参数类型必须是int(其实只会用到低八位)
public void write(byte[] b) throws IOException; //批量写入，一次性写入数组b的长度个字节
public int read(byte[] b, int off, int len); //批量读取的一个重载方法
```

## FileInputStream/FileOutputStream

FileInputStream的两个较为常用的构造方法

```
public FileInputStream(File file, boolean append) throws FileNotFoundException //其中append为true表明追加，为false表明覆盖
public FileInputStream(String filePath) throws FileNotFoundException
```

用法如下

```
import java.io.File;
import java.io.FileInputStream;
import java.io.InputStream;

public class StreamTest {

    public static void main(String[] args) {
        int data = 0;
        int index = 0;
        byte[] buffer = new byte[1024];
        File file = null;
        InputStream in = null;
        try {
            file = new File("C:\\Users\\Administrator\\Desktop\\Docs\\test.txt");
            in = new FileInputStream(file);
            while ((data = in.read()) != -1) {
                buffer[index++] = (byte)data;
            }
            System.out.println(new String(buffer, 0, index, "utf-8"));
            in.close();
        } catch(Exception e) {
            e.printStackTrace();
        } finally {
        }
    }

}
```

写入文件的代码为

```
import java.io.File;
import java.io.FileOutputStream;
import java.io.OutputStream;
import java.nio.charset.Charset;

public class StreamTest {

    public static void main(String[] args) {
        File file = null;
        OutputStream out = null;
        try {
            file = new File("C:\\Users\\Administrator\\Desktop\\Docs\\test.txt");
            out = new FileOutputStream(file, true); //追加
            byte[] data = "Hello, world".getBytes(Charset.forName("utf-8"));
            out.write(data);
            out.close();
        } catch(Exception e) {
            e.printStackTrace();
        } finally {
        }
    }

}
```

## DataInputStream/DataOutputStream

这两个是装饰类，他们接受一个已有的InputStream/OutputStream为构造方法的参数，通过他们写入或读取数据时，可以读取或写入指定类型的数据，类型如下

```
read/writeInt：读取或写入四个字节，先是高字节再是低字节；
read/writeBoolean：写入一个字节，若值为true则写入1否则写入0
read/writeUTF：将字符串按utf-8编码写入。
...
```

需要注意的是这两个类写入的文件都是对应的二进制字节，也就是说打开文件是乱码的，但是读写均正常

```
import java.io.DataInputStream;
import java.io.DataOutputStream;
import java.io.File;
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

public class StreamTest {

    public static void main(String[] args) {
        try {
            File file = new File("C:\\Users\\Administrator\\Desktop\\Docs\\test.txt");
            DataOutputStream out = new DataOutputStream(new FileOutputStream(file));
            List<Student> list = Arrays.asList(new Student[] {
                    new Student(18, "张三", 80.9d), new Student(17, "李四", 67.5d)
            });
            out.writeInt(list.size());
            for (Student s : list) {
                out.writeInt(s.getAge());
                out.writeUTF(s.getName());
                out.writeDouble(s.getScore());
            }
            out.close();
            
            List<Student> students = new ArrayList<Student>();
            DataInputStream in = new DataInputStream(new FileInputStream(file));
            int size = in.readInt();
            for (int i = 0; i < size; i++) {
                Student s = new Student();
                s.setAge(in.readInt());
                s.setName(in.readUTF());
                s.setScore(in.readDouble());
                students.add(s);
            }
            for(Student s : students) {
                System.out.println(s.getAge());
                System.out.println(s.getName());
                System.out.println(s.getScore());
            }
            in.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    
    public static class Student {
        int age;
        String name;
        double score;
        public Student() {
            
        }
        public Student(int age, String name, double score) {
            this.age = age;
            this.name = name;
            this.score = score;
        }
        public void setAge(int age) {
            this.age = age;
        }
        public void setName(String name) {
            this.name = name;
        }
        public void setScore(double score) {
            this.score = score;
        }
        public int getAge() {
            return age;
        }
        public String getName() {
            return name;
        }
        public double getScore() {
            return score;
        }
    }

}
```

写入的文件内容为

```
       寮犱笁@T9櫃櫃?    鏉庡洓@P?  
```

读出的文件内容为

```
18
张三
80.9
17
李四
67.5 
```

## BufferedInputStream/BufferedOutputStream

FileInputStream/FileOutputStream没有缓冲，是按字节读取的，性能低，虽然可以按字节数组读取以提高性能，但是有时必须按字节读写，如果使用缓冲区的话就能很好的解决这个问题。

使用BufferedInputStream/BufferedOutputStream时，只需要将FileInputStream/FileOutputStream对象作为参数传入构造方法即可。如下

```
import java.io.BufferedInputStream;
import java.io.BufferedOutputStream;
import java.io.File;
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.nio.charset.Charset;

public class StreamTest {

    public static void main(String[] args) {
        try {
            File file = new File("C:\\Users\\Administrator\\Desktop\\Docs\\test.txt");
            BufferedOutputStream bufferedOut = new BufferedOutputStream(new FileOutputStream(file));
            bufferedOut.write("Feily Zhang\nHello , world".getBytes(Charset.forName("utf8")));
            bufferedOut.close();
            int size = 0;
            byte[] buffer = new byte[1024];
            BufferedInputStream bufferedIn = new BufferedInputStream(new FileInputStream(file));
            size = bufferedIn.read(buffer);
            System.out.println(new String(buffer, 0, size, "utf-8"));
            bufferedIn.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

结果为

```
Feily Zhang
Hello , world
```