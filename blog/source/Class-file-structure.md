title: "类文件结构"
date: 2019-04-22 15:43:25 +0800
update: 2019-04-22 15:43:25 +0800
author: me
cover: "-/images/class-file.png"
tags:
    - Java
preview: 名称描述该类型(连续地址空间)中的内容代表的是什么东西(魔数还是版本号？常量池？等等)，数量描述的是该名称内部的子项目有多少个。

---

Class文件是一组以8位字节为基础单位的二进制流，各个数据项目严格按照顺序紧凑的排列在Class文件之中，中间没有任何分隔符，这使得整个Class文件中存储的内容几乎全部都是程序运行的必要数据，没有空隙存在。当遇到需要占用8为字节以上空间的数据项时，则会按照高位在前的方式分割成若干个8位字节进行存储。

根据Java虚拟机规范的规定，Class文件格式采用一种类似于C语言结构体的伪结构来存储，这种伪结构只有两种数据类型：无符号数和表。

无符号数属于基本数据类型，以u1、u2、u4、u8来分别代表1、2、4、8个字节的无符号数，无符号数可以描述数字、索引引用、数量值或者按照UTF-8编码构成的字符串值。

表是多个无符号数或其他表作为数据项构成的复合数据类型，所有表都习惯性地以“_info”结尾，表用于描述有层次关系的复合结构的数据，整个Class文件本质上就是一张表。Class文件由以下表项构成(需要注意的是，**数据类型只是指明了该名称表项占用的连续内存的多少**)

|类型|名称|数量|
|------|------|------|
|u4|magic|1|
|u2|minor_version|1|
|u2|major_version|1|
|u2|constant_pool_count|1|
|cp_info|constant_pool|constant_pool_count - 1|
|u2|access_flags|1|
|u2|this_class|1|
|u2|super_class|1|
|u2|interfaces_count|1|
|u2|interfaces|interfaces_count|
|u2|fields_count|1|
|field_info|fields|fields_count|
|u2|methods_count|1|
|method_info|methods|methods_count|
|u2|attributes_count|1|
|attribute_info|attributes|attributes_count|

无论是无符号数还是表，当需要描述同一类型但数量不定的多个数据时，经常会使用一个前置的容量计数器加若干个连续的数据项的形式，这时候称这一系列连续的某一类型的数据为某一类型的集合。

## 一、魔数与Class文件的版本

魔数的唯一作用是确定这个文件是否为一个能被虚拟机接受的Class文件。Java文件的魔数是十六进制的`CAFEBABE`，攻占用四个字节，即名称为魔数的表项的类型为u4，那么从Class文件中开头连续数四个字节的数据就是魔数，如下

![](/images/article/class1.png)

紧接着魔数的四个字节分别是此版本号和主版本号，分别占两个字节，共4个字节，如下

![](/images/article/class2.png)

主版本号为52，说明是JDK1.8编译出来的。JDK1.1的主版本号为45，1.2的主版本号为46，1.7的主版本号为51，自然地1.8的主版本号为52.

## 二、常量池容量计数与常量池

常量池容量计数（constant_pool_count）是一个u2类型的数据，占用16个bit，即最多表示65536个(0 - 65535)个常量池项目，即常量池最多有65536个项目。

![](/images/article/class3.png)

可见，该Class文件中常量池容量计数值为19，即常量池中共有18项常量(该计数是从1开始的而非0，索引值为1 - 18)，也就意味着接下来的类型为`cp_info`的`constant_pool`会有`constant_pool_count - 1`项常量。

常量池`constant_pool`是Class文件表中的一张子表，主要存放两大类常量：字面量和符号引用。字面量包括文本字符串、被声明为final的常量值等。而符号引用则包括类和接口的全限定名、字段的名称和描述符、方法的名称和描述符。完整结构如下

<table>
  <tr>
    <td>常量</td><td>项目</td><td>类型</td><td>对应描述</td><td>标志</td><td>常量描述</td>
  </tr>
  <tr>
    <td rowspan = 3>CONSTANT_Utf8_info</td><td>tag</td><td>u1</td><td>同[标志]，为1</td><td rowspan = 3>1</td><td rowspan = 3>UTF-8编码的字符串</td>
  </tr>
  <tr>
    <td>length</td><td>u2</td><td>UTF-8编码的字符串占用的字节数</td>
  </tr>
  <tr>
    <td>bytes</td><td>u1</td><td>长度为length的UTF-8编码的字符串</td>
  </tr>
  
  <tr>
    <td rowspan = 2>CONSTANT_Integer_info</td><td>tag</td><td>u1</td><td>同[标志]，为3</td><td rowspan = 2>3</td><td rowspan = 2>整型字面量</td>
  </tr>
  <tr>
    <td>bytes</td><td>u4</td><td>按照高位在前存储的int值</td>
  </tr>
  
  <tr>
    <td rowspan = 2>CONSTANT_Float_info</td><td>tag</td><td>u1</td><td>同[标志]，为4</td><td rowspan = 2>4</td><td rowspan = 2>浮点型字面量</td>
  </tr>
  <tr>
    <td>bytes</td><td>u4</td><td>按照高位在前存储的float值</td>
  </tr>
  
  <tr>
    <td rowspan = 2>CONSTANT_Long_info</td><td>tag</td><td>u1</td><td>同[标志]，为5</td><td rowspan = 2>5</td><td rowspan = 2>长整型型字面量</td>
  </tr>
  <tr>
    <td>bytes</td><td>u8</td><td>按照高位在前存储的Long值</td>
  </tr>
  
  <tr>
    <td rowspan = 2>CONSTANT_Double_info</td><td>tag</td><td>u1</td><td>同[标志]，为6</td><td rowspan = 2>6</td><td rowspan = 2>双精度浮点型字面量</td>
  </tr>
  <tr>
    <td>bytes</td><td>u8</td><td>按照高位在前存储的double值</td>
  </tr>
  
  <tr>
    <td rowspan = 2>CONSTANT_Class_info</td><td>tag</td><td>u1</td><td>同[标志]，为7</td><td rowspan = 2>7</td><td rowspan = 2>类或接口的符号引用</td>
  </tr>
  <tr>
    <td>index</td><td>u2</td><td>指向全限定名常量项的索引，即指向常量池第index项常量，该常量类型为CONSTANT_Utf8_info</td>
  </tr>
  
  <tr>
    <td rowspan = 2>CONSTANT_String_info</td><td>tag</td><td>u1</td><td>同[标志]，为8</td><td rowspan = 2>8</td><td rowspan = 2>字符串类型字面量</td>
  </tr>
  <tr>
    <td>index</td><td>u2</td><td>指向字符串字面量的索引，即指向常量池第index项常量，该常量类型为CONSTANT_Utf8_info</td>
  </tr>
  
  <tr>
    <td rowspan = 3>CONSTANT_Fieldref_info</td><td>tag</td><td>u1</td><td>同[标志]，为9</td><td rowspan = 3>9</td><td rowspan = 3>字段的符号引用</td>
  </tr>
  <tr>
    <td>index</td><td>u2</td><td>指向声明字段的类或接口描述符CONSTANT_Class_info的索引项，即第index项常量，该常量类型为CONSTANT_Class_info</td>
  </tr>
  <tr>
    <td>index</td><td>u2</td><td>指向字段描述符CONSTANT_NameAndType的索引项，即第index项常量，该常量类型为CONSTANT_NameAndType</td>
  </tr>
  
  <tr>
    <td rowspan = 3>CONSTANT_Methodref_info</td><td>tag</td><td>u1</td><td>同[标志]，为10</td><td rowspan = 3>10</td><td rowspan = 3>类中方法的符号引用</td>
  </tr>
  <tr>
    <td>index</td><td>u2</td><td>指向声明方法的类描述符CONSTANT_Class_info的索引项，即第index项常量，该常量类型为CONSTANT_Class_info</td>
  </tr>
  <tr>
    <td>index</td><td>u2</td><td>指向名称及类型描述符CONSTANT_NameAndType的索引项，即第index项常量，该常量类型为CONSTANT_NameAndType</td>
  </tr>
  
  <tr>
    <td rowspan = 3>CONSTANT_InterfaceMethodref_info</td><td>tag</td><td>u1</td><td>同[标志]，为11</td><td rowspan = 3>11</td><td rowspan = 3>接口中方法的符号引用</td>
  </tr>
  <tr>
    <td>index</td><td>u2</td><td>指向声明方法的接口类描述符CONSTANT_Class_info的索引项，即第index项常量，该常量类型为CONSTANT_Class_info</td>
  </tr>
  <tr>
    <td>index</td><td>u2</td><td>指向名称及类型描述符CONSTANT_NameAndType的索引项，即第index项常量，该常量类型为CONSTANT_NameAndType</td>
  </tr>
  
  <tr>
    <td rowspan = 3>CONSTANT_NameAndType_info</td><td>tag</td><td>u1</td><td>同[标志]，为12</td><td rowspan = 3>12</td><td rowspan = 3>字段或方法的部分符号引用</td>
  </tr>
  <tr>
    <td>index</td><td>u2</td><td>指向该字段或方法名称常量项的索引</td>
  </tr>
  <tr>
    <td>index</td><td>u2</td><td>指向该字段或方法描述符常量项的索引</td>
  </tr>
</table>

由于常量池每一项的第一个子项均为tag，因此可以通过tag的数值区分该项常量为常量池中的哪个项目，再通过该项目子项的项目及其类型就可以确定下一个常量池的tag。

举个例子，在本例中常量池的第一个u1类型的tag为十进制10，如下

![](/images/article/class4.png)

即代表第一项常量为**类中方法的符号引用**，该项常量除tag外各有两个字节的index，第一个index值为

![](/images/article/class5.png)

即代表指向常量池中第4项常量，第二个index为

![](/images/article/class6.png)

即代表指向常量池中第15项常量。接下来第二项常量为

![](/images/article/class7.png)

tag为9，代表这项常量是**字段的符号引用**，两个u2类型的index的值分别为

![](/images/article/class8.png)

![](/images/article/class9.png)

即分别指向第3项和第16项常量。

剩余常量的分析与上述同理，即都是通过tag判断常量池中常量类型，然后tag后该常量池经过子项的数据类型代表的连续地址空间之后就是另一个常量的tag，继续重复即可。

剩余的常量我们借助Class文件字节码分析工具javap来完成，常量池中完整内容如下

```
C:\Users\Administrator>javap -verbose C:\Users\Administrator\Desktop\docs\TestCl
ass.class
Classfile /C:/Users/Administrator/Desktop/docs/TestClass.class
  Last modified 2019-4-22; size 295 bytes
  MD5 checksum 81f2ab948a7a3068839b61a8f91f634b
  Compiled from "TestClass.java"
public class org.fenixsoft.clazz.TestClass
  minor version: 0
  major version: 52
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref          #4.#15         // java/lang/Object."<init>":()V
   #2 = Fieldref           #3.#16         // org/fenixsoft/clazz/TestClass.m:I
   #3 = Class              #17            // org/fenixsoft/clazz/TestClass
   #4 = Class              #18            // java/lang/Object
   #5 = Utf8               m
   #6 = Utf8               I
   #7 = Utf8               <init>
   #8 = Utf8               ()V
   #9 = Utf8               Code
  #10 = Utf8               LineNumberTable
  #11 = Utf8               inc
  #12 = Utf8               ()I
  #13 = Utf8               SourceFile
  #14 = Utf8               TestClass.java
  #15 = NameAndType        #7:#8          // "<init>":()V
  #16 = NameAndType        #5:#6          // m:I
  #17 = Utf8               org/fenixsoft/clazz/TestClass
  #18 = Utf8               java/lang/Object
SourceFile: "TestClass.java"
```

可见前两项常量的分析我们是正确的，常量池中共有18项常量，第一项常量的具体子项分别指向第4和第15项常量，内容分别为`java/lang/Object`和`"<init>":()V`；

而第二项常量分别指向第3和16项常量，内容分别为`org/fenixsoft/clazz/TestClass`和`m:I`。

## 三、访问标志

![](/images/article/class10.png)

常量池结束后的两个字节代表访问标志(access_flags)，该标志用于识别一些类或接口层次的访问信息，包括：这个Class是类还是接口；是否定义为public类型；是否定义为abstract类型；如果是的话，是否被声明为final，等等。具体的标志位以及含义见下表

