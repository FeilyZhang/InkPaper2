title: "再看Java文件操作之字符流"
date: 2018-12-18 11:50:49 +0800
update: 2018-12-18 11:50:49 +0800
author: me
cover: "-/images/java.jpg"
tags:
    - Java Core
preview: 字符流以字符为单位操作文件。

---

字符流以字符为单位操作文件，以下是Java中字符流的主要类

+ Reader/Writer：字符流的基类，是抽象类；
+ FileRead/FileWrite：输入源与输出源均为文件的字符流；
+ InputStreamReader/OutputStreamWriter：适配器类，将字节流转换为字符流；
+ CharArrayReader/CharArrayWriter：属于源与输出源是char数组的字符流；
+ StringReader/StringWriter：输入源与输出源为String的字符流；
+ BufferedReader/BufferedWriter：装饰类，对输入/输出提供缓冲，以及按行读写功能；
+ PrintWriter：装饰类，可将基本类型与对象类型转换为其字符串形式输出的类。

## Reader类的主要方法如下

```
public int read() throws IOException
public int read(char cbuf[]) throws IOException
abstract public void close() throws IOException
```

## Writer类的主要方法如下

```
public void writer(int c)
public void writer(char cbuf)
public void writer(String str) throws IOException
abstract public void flush() throws IOException
```

## InputStreamReader/OutputStreamWriter

InputStreamReader/OutputStreamWriter是适配器类，能够将InputStream/OutputStream转换为Reader/Writer，使用方法如下

```
import java.io.File;
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.OutputStreamWriter;
import java.io.Reader;
import java.io.Writer;

public class StreamTest {

    public static void main(String[] args) {
        try {
            File file = new File("C:\\Users\\Administrator\\Desktop\\Docs\\test.txt");
            Writer writer = new OutputStreamWriter(new FileOutputStream(file));
            writer.write("Feily Zhang");
            writer.close();
            Reader reader = new InputStreamReader(new FileInputStream(file));
            char[] data = new char[1024];
            int size = reader.read(data);
            System.out.println(new String(data, 0, size));
			reader.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

## FileRead/FileWrite

FileRead/FileWrite的用法类似于FileInputStream/FileOutputStream，直接传入文件名就可以按字符流读取，如下

```
import java.io.File;
import java.io.FileReader;
import java.io.FileWriter;
import java.io.IOException;
import java.io.Reader;
import java.io.Writer;

public class StreamTest {

    public static void main(String[] args) {
        try {
            File file = new File("C:\\Users\\Administrator\\Desktop\\Docs\\test.txt");
            Writer writer = new FileWriter(file);
            writer.write("Hello, world");
            writer.close();
            Reader reader = new FileReader(file);
            char[] data = new char[1024];
            int size = reader.read(data);
            System.out.println(new String(data, 0, size));
            reader.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

## BufferedReader/BufferedWriter

BufferedReader/BufferedWriter为Reader/Writer提供缓冲区功能，属于装饰类，能按行读写，示例如下

```
import java.io.BufferedReader;
import java.io.BufferedWriter;
import java.io.File;
import java.io.FileReader;
import java.io.FileWriter;
import java.io.IOException;

public class StreamTest {

    public static void main(String[] args) {
        try {
            File file = new File("C:\\Users\\Administrator\\Desktop\\Docs\\test.txt");
            BufferedWriter writer = new BufferedWriter(new FileWriter(file));
            writer.write("Hello, Feily Zhang\nHello, World");
            writer.close();
            BufferedReader read = new BufferedReader(new FileReader(file));
            char[] data = new char[1024];
            int size = read.read(data);
            System.out.println(new String(data, 0, size));
            read.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```