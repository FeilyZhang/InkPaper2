title: "Class文件结构"
date: 2019-04-22 15:43:25 +0800
update: 2019-04-22 21:24:05 +0800
author: me
cover: "-/images/class-file.png"
tags:
    - Java
preview: Class文件被抽象组织为了一张表，名称描述该类型(连续地址空间)中的内容代表的是什么东西(魔数还是版本号？常量池？等等)，数量描述的是该名称内部的子项目有多少个。

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

魔数的唯一作用是确定这个文件是否为一个能被虚拟机接受的Class文件。Java文件的魔数是十六进制的`CAFEBABE`，共占用四个字节，即名称为魔数的表项的类型为u4，那么从Class文件中开头连续数四个字节的数据就是魔数，如下

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

常量池结束后的两个字节代表访问标志(access_flags)，该标志用于识别一些类或接口层次的访问信息，包括：这个Class是类还是接口；是否定义为public类型；是否定义为abstract类型；如果是的话，是否被声明为final，等等。具体的标志位以及含义见下表

|标志名称|标志值|含义|
|------|------|------|
|ACC_PUBLIC|0x0001|是否为public类型|
|ACC_FINAL|0x0010|是否被声明为final，只有类可设置|
|ACC_SUPER|0x0020|是否允许使用`invokespecial`字节码指令，JDK1.2之后编译出来的类的这个标志为真|
|ACC_INTERFACE|0x0200|标识这是一个接口|
|ACC_ABSTRACT|0x0400|是否为abstract类型，对于接口或者抽象类来说，此标志值为真，其它值为假|
|ACC_SYNTHETIC|0x1000|标识这个类并非由用户代码产生的|
|ACC_ANNOTATION|0x2000|标识这是一个注解|
|ACC_ENUM|0x4000|标识这是一个枚举|

access_flags共有16个标志位可用，当前只定义了8个，没有使用到的标志位要求一律为0。以本例为例，TestClass被public关键字修饰但是并没有被声明为final和abstract，并且使用了JDK1.2之后的编译器进行编译，因此access_flags的值应为：`0x0001 | 0x0020 = 0x0021`,如下图所示

![](/images/article/class10.png)

## 三、类索引、父类索引与接口索引集合

类索引(this_class)与父类索引(super_class)均为u2类型的数据，其值分别指向常量池中的某项常量，即符号引用。类索引用于确定这个类的全限定名，父类索引用于确定这个类的父类的全限定名。由于Java不允许多重继承，所以父类索引只有一个，除了java.lang.Object之外，所有的Java类都有父类，也就是说出了java.lang.Object之外，所有的Java类的父类索引(super_class)都不为0。在本例中如下图所示

![](/images/article/class11.png)

即类索引指向常量池中第3项常量，值为`org/fenixsoft/clazz/TestClass`，即该类的全限定名。

![](/images/article/class12.png)

父类的全限定名指向常量池第四项常量，为`java/lang/Object`。

接口索引集合的计数器`interfaces_count`为0，代表接口索引集合`interfaces`字段无值，直接跳过(如下图)。接口索引集合用来描述这个类实现了那些接口，这些被实现的接口按照implements的顺序从做左到右的顺序排列在接口索引集合中。接口索引集合的元素仍然是u2类型的数据，即指向常量池中的某项常量，该常量给出了接口的全限定名，接口索引集合计数器为多少就连续数多少个u2类型数据。如果当前类本身就是一个接口，那么接口索引集合后的值就是extends语句。

![](/images/article/class13.png)

## 四、字段表集合

字段表(field_info)用来描述接口或者类中声明的变量。字段(field)包括了类级变量或实例级变量，但不包括在方法内部声明的变量(这种变量包含在栈中的局部变量表中)。一个字段的描述信息都包括：字段的作用域(public、private、protected修饰符)、是实例变量还是类变量(static修饰符)、可变性(final)、并发可见性(volatile修饰符，是否强制从主内存读写)、可否序列化(transient修饰符)、字段数据类型(基本类型、对象、数组)、字段名称。

这些信息中，各个修饰符都是布尔值，即要么有要么没有，很适合用标志位来表示。而字段叫什么名字、字段被定义为什么数据类型是无法固定的，也就意味着会指向常量池中的某项常量。字段表是Class文件表的一张子表，其格式如下


|类型|名称|数量|描述|
|------|------|------|------|
|u2|access_flags|1|字段的修饰符，具体值见下表|
|u2|name_index|1|字段名称|
|u2|descriptor_index|1|字段数据类型修饰符,各数据类型的常量表示见下下表|
|u2|attributes_count|1|该字段与下一个字段描述字段表集合的额外信息|
|attribute_info|attributes|attributes_count|该字段与上一个字段描述字段表集合的额外信息|

字段访问标志access_flags的标志位与值如下表


|标志名称|标志值|含义|
|------|------|------|
|ACC_PUBLIC|0x0001|字段是否为public|
|ACC_PRIVATE|0x0002|字段是否为private|
|ACC_PROTECTED|0x0004|字段是否为protected|
|ACC_STATIC|0x0008|字段是否为static|
|ACC_FINAL|0x0010|字段是否为final|
|ACC_VOLATILE|0x0040|字段是否为volatile|
|ACC_TRANSIENT|0x0080|字段是否为transient|
|ACC_SYNTHETIC|0x1000|字段是否由编译器自动产生的|
|ACC_ENUM|0x4000|字段是否为enum|

常量池中描述符表示字符的含义为

|标识字符|含义|
|------|------|
|B|基本类型byte|
|C|基本类型char|
|D|基本类型double|
|F|基本类型float|
|I|基本类型int|
|J|基本类型long|
|S|基本类型short|
|Z|基本类型boolean|
|V|特殊类型void|
|L|对象类型，如Ljava/lang/Object;|

对于数组类型，每一维度将使用一个前置的`[`字符来描述，如一个定义为`java.lang.String[][]`类型的二维数组，将被记录为：`[[Ljava/lang/String;`，其中`[[`二维数组，`L`表示这是一个对象，`java/lang/String`表示这是一个String型。合起来就是一个String对象二维数组，那么一个整型数组`int[]`会被记录为`[I`。

题外话：而用描述符来描述方法时，按照先参数列表，后返回值的顺序描述，参数列表按照参数的严格顺序放在一组小括号`()`之内。如方法`void inc()`的描述符为`()V`，方法`java.lang.String toString()`的描述符为`()Ljava/lang/String;`,表示该方法无参，返回值为对象`(L)String`。而方法`int indexOf(char[] source, int sourceOffset, int sourceCount, char[] target, int targetOffset, int targetCount, int fromIndex)`的描述符为`([CII[CIII)I`,很好理解，不解释。

接下来对TestClass字段表集合的分析如下

![](/images/article/class14.png)

首先该字段表集合计数器fields_count为1，即字段集合表中只有一个实例变量或者类变量。那么进入字段集合表继续分析这一个变量。

![](/images/article/class15.png)

上图显示该字段的access_flags值为`0x0002`，查字段访问标志表得，该字段的访问修饰符为`private`。

![](/images/article/class16.png)

上图显示，`name_index`字段指向了常量池中第五项常量，内容为

```
#5 = Utf8               m
```

即该字段的名称为`m`。

![](/images/article/class17.png)

上图中descriptor_index值为6，该值仍然指向常量池，内容为

```
#6 = Utf8               I
```

那么综上所述就可以推断出该字段被定义为`private int m;`。其后的两个字节表示attributes_count，如下图

![](/images/article/class18.png)

值为0，代表attributes无值，可以直接跳过。

## 五、方法表集合

方法表集合与字段表集合的解释几乎一样，连结构也一样，如下表所示

|类型|名称|数量|描述|
|------|------|------|------|
|u2|access_flags|1|方法的修饰符，具体值见下表|
|u2|name_index|1|方法名称|
|u2|descriptor_index|1|方法返回值修饰符,同字段表返回值修饰符|
|u2|attributes_count|1|该字段与下一个字段描述方法表集合的额外信息|
|attribute_info|attributes|attributes_count|该字段与上一个字段描述方法表集合的额外信息|


方法访问标志access_flags如下表

|标志名称|标志值|含义|
|------|------|------|
|ACC_PUBLIC|0x0001|方法是否为public|
|ACC_PRIVATE|0x0002|方法是否为private|
|ACC_PROTECTED|0x0004|方法是否为protected|
|ACC_STATIC|0x0008|方法是否为static|
|ACC_FINAL|0x0010|方法是否为final|
|ACC_SYNCHRONIZED|0x0020|方法是否为synchronized|
|ACC_BRIDGE|0x0040|方法是否是由编译器产生的桥接方法|
|ACC_VARARGS|0x0080|方法是否接受不定参数|
|ACC_NATIVE|0x0100|方法是否为native|
|ACC_ABSTRACT|0x0400|方法是否为abstract|
|ACC_STRICT|0x0800|方法是否为strictfp|
|ACC_SYNTHETIC|0x1000|方法是否是由编译器自动产生的|

接下来分析TestClass

![](/images/article/class19.png)

methods_count为2，代表该类有两个方法(分别是编译器添加的构造器`<init>`和源码中的`inc()`方法)。先看第一个方法的access_flags

![](/images/article/class20.png)

值为0x0001，查表得该方法的修饰符为public，name_index如下图

![](/images/article/class21.png)

可见指向常量池中第7项常量，内容为

```
#7 = Utf8               <init>
```

可见该方法为构造器`<init>`，接下来看descriptor_index的值，如下图

![](/images/article/class22.png)

值为8，仍然是指向常量池，第八项常量为

```
#8 = Utf8               ()V
```

即无参无返回值。接下来看attributes_count，如下图

![](/images/article/class23.png)

值为1，代表方法表中的属性表集合中有一项属性。接下来看Class文件的最后一个项目——属性表集合(attribute_info)。

## 六、属性表集合

Class文件、字段表、方法表均可以携带自己的属性表集合，以用于描述某些场景专有的信息。与Class文件中其他的数据项目要求严格的顺序、长度和内容不同，属性表集合的限制稍微宽松了一些，不再要求各个属性表具有严格的顺序，并且只要不与已有的属性名重复，任何人实现的编译器都可以向属性表中写入自己定义的属性信息，Java虚拟机运行时会忽略掉它不认识的属性。为了能正确解析Class文件，《Java虚拟机规范（第2版）》中预定义了9项虚拟机应当能识别的属性，如下

|属性名称|使用位置|含义|
|------|------|------|
|Code|方法表|Java代码编译成的字节码指令|
|ConstantValue|字段表|final关键字定义的常量值|
|Deprecated|类、方法表、字段表|被声明为deprecated的方法和字段|
|Exceptions|方法表|方法抛出的异常|
|InnerClasses|类文件|内部类列表|
|LineNumberTable|上述Code属性中|Java源码的行号与字节码指令的对应关系|
|LocalVariableTable|上述Code属性中|方法的局部变量描述|
|SourceFile|类文件|源文件名称|
|Synthetic|类、方法表、字段表|标识方法或字段是否为编译器自动生成的|

对于每个属性，它的名称需要从常量池中引用一个CONSTANT_Utf8_info类型的常量来表示，而属性值的结构则是完全自定义的，只需要说明属性值所占用的位数长度即可。一个符合规则的属性表应该满足如下定义的结构(作用就是字段表和方法表中attributes_count指明了额外的属性信息后进入属性表集合中根据attribute_name_index查常量池从而确定为何种属性)

|类型|名称|数量|
|------|------|------|
|u2|attribute_name_index|1|
|u2|attribute_length|1|
|u1|info|attribute_length|

### 6.1 Code属性

对于方法表集合中的属性表集合，Code属性是其中的一个。Java程序方法体里面的代码经过javac编译器处理之后，最终会变成字节码指令存储在方法表集合的属性表集合的Code属性中，也就是说方法表集合属性表集合之前的一些子项只是方法名称、返回值、修饰符的一些描述，真正的方法代码就在方法表集合的属性表集合的Code属性中，Code属性的表结构如下

|类型|名称|数量|
|------|------|------|
|u2|attribute_name_index|1|
|u4|attribute_length|1|
|u2|max_stack|1|
|u2|max_locals|1|
|u4|code_length|1|
|u1|code|code_length(即每个字节码占用1个字节，code_length为多大，那么该方法就有多少个code，连续读)|
|u2|exception_table_length|1|
|exception_info|exception_table|exception_table_length|
|u2|attributes_count|1|
|attribute_info|attributes|attributes_count|

attribute_name_index指向CONSTANT_Utf8_info型常量的索引，常量值固定为`Code`，它代表了该属性的属性名称，attribute_length指示了属性**值**的长度，由于属性名称索引与属性长度共6个字节，所以属性值的长度固定为整个属性表的长度减去6个字节。

max_stack代表了操作数栈深度的最大值，在方法执行的任意时刻，操作数栈都不会超过这个深度。虚拟机在运行的时候需要根据这个值来分配栈帧(Frame)中的操作栈深度。

max_locals代表了局部变量表所需的存储空间，基本单位是Slot，Slot是虚拟机为局部变量分配内存所使用的最小单位。对于byte、char、float、int、short、boolean、reference和returnAddress等长度不超过32位的数据类型，每个局部变量占用1个Slot，而double和long这两种64位的数据类型则需要2个Slot来存放。**方法参数（包括实例方法中的隐藏参数this）、显式异常处理器的参数(Exception Handler Parameter,即try-catch语句中catch快所定义的异常)、方法体中定义的局部变量都需要使用局部变量表来存放。另外，并不是方法中用到了多少个局部变量就把局部变量所占的Slot之和作为max_locals的值，原因是局部变量表中的Slot可以重用，当代码执行超出一个局部变量的作用域时，这个局部变量所占的Slot就可以被其他局部变量所占用，编译器会根据变量的作用域来分类Slot并分配给各个变量使用，然后计算出max_locals的大小。**

code_length和code用来存储Java源程序编译后生成的字节码指令，code_length代表字节码的长度，code是用于存储字节码指令的一系列字节流。既然名为字节码指令，那么每个指令就是一个u1类型的单字节，当虚拟机读取到code中的一个字节码时，就可以相应地找出这个字节码代表什么指令，并且可以知道这条指令后面是否需要跟随参数以及参数应该如何理解。我们知道一个u1类型的取值范围为0 - 255，也就是一共可以表达256条指令。当前，Java虚拟机规范中定义了约200条编码值对应的指令含义。

关于code_length,虽然是一个u4类型的长度，理论上可以取值2<sup>32</sup> - 1,也就是说理论上一个方法可以有2<sup>32</sup> - 1条**指令**，但是虚拟机规范明确指出，一个方法不允许超过65535条**字节码指令**，如果超过了这个限制，javac编译器就会拒绝编译。

**Code属性是Class文件中最重要的一个属性，如果把一个Java程序中的信息分为代码(Code,方法体里面的Java代码)和元数据(Metadata，包括类、字段、方法定义及其他信息)两部分，那么在整个Class文件里，Code属性用于描述代码，所有的其他数据项目都用于描述元数据。**

继续以TestClass为例，说明方法表集合中属性表集合的Code属性

![](/images/article/class23.png)

由上图得知，方法表集合中第一个方法`<init>`attributes_count的值为1,即代表该方法包含一个额外的描述信息，进入属性表集合，根据属性表结构第一项为u2类型的attribute_name_index，那么先连续读2个字节确定该属性属于何种属性

![](/images/article/class24.png)

得知指向常量池第9项常量，内容为

```
#9 = Utf8               Code
```

那么就可以确定方法表集合中的属性表集合中的属性为Code属性，即方法体中经编译产生的字节码，那就查Code属性结构表继续分析，连续四个字节为attribute_length

![](/images/article/class25.png)

即长度为29。如下两图分别指出了max_stack与max_locals的值

![](/images/article/class26.png)

![](/images/article/class27.png)

分别为1。接下来连续四个字节为code_length,即字节码指令的长度，如下

![](/images/article/class28.png)

值为5，那么连续读5个字节就是字节码指令(code),分别如下

![](/images/article/class29.png)

此处的查表查的的编码值与字节码指令的对应表，这里无法列出。仅作说明

+ 读入2A，查表得0x2A所对应的指令为aload_0,含义是将第0个Slot中卫reference类型的本地变量推送至栈顶；
+ 读入B7，查表得0xB7所对应的指令为invokespecial，作用是以栈顶的reference类型的数据所指向的对象作为方法的接受者，调用此对象的实例构造器方法、private方法或其父类的方法。这个方法有一个u2类型的参数说明具体调用哪一个方法，它指向常量池中的一个CONSTANT_Methodref_info类型的常量，即此方法的方法符号引用；
+ 读入u2类型的数据00 01，这是invokespacial的参数，查常量池0x000A对应的常量为实例构造器`<init>`方法的符号引用；
+ 读入B1，查表得0xB1对应的指令为return，含义是返回此方法，并且返回值为void。这条指令执行后，当前方法结束。

从这里可以看出，方法的执行过程中的数据交换、方法调用等操作都是基于栈(操作栈)进行的。因此可以初步猜测：Java虚拟机执行字节码是基于栈的体系结构。

剩下个另一个方法通过javap命令来计算，如下

```
{
  public org.fenixsoft.clazz.TestClass();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>
":()V
         4: return
      LineNumberTable:
        line 3: 0

  public int inc();
    descriptor: ()I
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=1, args_size=1
         0: aload_0
         1: getfield      #2                  // Field m:I
         4: iconst_1
         5: iadd
         6: ireturn
      LineNumberTable:
        line 8: 0
}
SourceFile: "TestClass.java"
```

这是直接计算出了方法表集合中的所有信息，包括描述符标识字符descriptor、方法名称等、访问标志还有属性表集合Code属性。