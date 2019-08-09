title: "C++中的预定义数据类型"
date: 2019-08-07 23:13:42 +0800
update: 2019-08-07 23:13:42 +0800
author: me
cover: "-/images/c++.jpg"
tags:
    - C++
preview: C++中的预定义数据类型包括整型、浮点型、字符型、布尔型、空类型、指针类型, 自动存储、静态存储和动态存储。

---

## 一、整型

|类型|位数|
|---|---|
|short|至少16位|
|int|至少和short一样长|
|long|至少32位，且至少与int一样长|
|long long|至少64位，且至少与long一样长|

可见以上描述只是规定了各类型的最小长度，在不同的计算机上有不同的实现。查看本机类型的长度可以使用`sizeof`运算符，其得到的结果的单位为字节。如下所示

```
#include <iostream>

void showLimits();

int main() {
	showLimits();
	return 0;
}

void showLimits() {
	std::cout << "sizeof(short) = " << sizeof(short) << std::endl;
	std::cout << "sizeof(int) = " << sizeof(int) << std::endl;
	std::cout << "sizeof(long) = " << sizeof(long) << std::endl;
	std::cout << "sizeof(long long) = " << sizeof(long long) << std::endl;
	std::cin.get();
}
```

本机输出为

```
sizeof(short) = 2
sizeof(int) = 4
sizeof(long) = 4
sizeof(long long) = 8
```

也可以通过变量名来查看特定数据类型的长度，在这种情况下，`sizeof`运算符的括号可以忽略，如下

```
include <iostream>

void showLimits();

int main() {
	showLimits();
	return 0;
}

void showLimits() {
	short s = 1;
	int i = 2;
	long l = 3;
	long long ll = 4;
	std::cout << "sizeof s  = " << sizeof s << std::endl;
	std::cout << "sizeof i = " << sizeof i << std::endl;
	std::cout << "sizeof l = " << sizeof l << std::endl;
	std::cout << "sizeof ll = " << sizeof ll << std::endl;
	std::cin.get();
}
```

其次，头文件`climits`中包含了关于整型的限制信息，`climits`中的符号常量如下表所示

|符号常量|表示|
|-----|-----|
|CHAR_BIT|char的位数|
|CHAR_MAX|char的最大值|
|CHAR_MIN|char的最小值|
|SCHAR_MAX|signed char的最大值|
|SCHAR_MIN|signed char的最小值|
|UCHAR_MAX|unsigned char的最大值|
|SHRT_MAX|short的最大值|
|SHRT_MIN|short的最小值|
|USHRT_MAX|unsigned short的最大值|
|INT_MAX|int的最大值|
|INT_MIN|int的最小值|
|UNIT_MAX|unsigned int的最大值|
|LONG_MAX|long的最大值|
|LONG_MIN|long的最小值|
|ULONG_MAX|unsigned long的最大值|
|LLONG_MAX|long long的最大值|
|LLONG_MIN|long long的最小值|
|ULLONG_MAX|unsigned long的最大值|

部分示例程序如下

```
#include <iostream>
#include <climits>

void showClimits();

int main() {
	showClimits();
	return 0;
}

void showClimits() {
	std::cout << "SHRT_MAX = " << SHRT_MAX << std::endl;
	std::cout << "SHRT_MIN = " << SHRT_MIN << std::endl;
	std::cout << "INT_MAX = " << INT_MAX << std::endl;
	std::cout << "INT_MIN = " << INT_MIN << std::endl;
	std::cout << "LONG_MAX = " << LONG_MAX << std::endl;
	std::cout << "LONG_MIN = " << LONG_MIN << std::endl;
	std::cout << "LLONG_MAX = " << LLONG_MAX << std::endl;
	std::cout << "LLONG_MIN = " << LLONG_MIN << std::endl;
	std::cin.get();
}
```

输出为

```
SHRT_MAX = 32767
SHRT_MIN = -32768
INT_MAX = 2147483647
INT_MIN = -2147483648
LONG_MAX = 2147483647
LONG_MIN = -2147483648
LLONG_MAX = 9223372036854775807
LLONG_MIN = -9223372036854775808
```

对于无符号数而言，可以增大相应数据类型能够表示的最大值，这是通过牺牲掉负数实现的。对于有符号数和无符号数而言，一旦超出它所能表示的最大范围，就会发生溢出，具体溢出行为如下所示


## 二、浮点型

浮点数有两种表示方法，分别是标准小数点表示法和E表示法。E表示法适合表示非常大的数和非常小的数。浮点数类型如下

|类型|位数|
|-----|-----|
|float|至少32位|
|double|至少48位，且不少于float|
|long double|至少和double一样多|

C++中浮点数的默认类型为double，如果希望存储float类型，那么为小数加上f或F后缀，对于long double类型，需要加上l或L后缀。

浮点数的优点在于可以表示整数之间的值；表示的范围比较大；但是运算速度慢于整数，且精度会降低。

## 三、字符型

char用于处理字符，但也可以将它用作比short更短的整型，因为其底层存储的依旧是字符的编码，所以由于char是一种整型，那么也可以进行加法运算，这样就代表的是以原始编码为基址，以加数为偏移的字符编码。但是程序仍然会输出字符而不是编码这是由于cout与cin会完成转换工作，要查看其编码，可以将char强制转换为整型即可。char占8位。示例程序如下

```
#include <iostream>

int main() {
	char c = 'M';
	std::cout << "c is " << c << " and c's code is " << (int)c << std::endl;
	c += 1;
	std::cout << "now c is " << c << " and c's code is " << (int)c << std::endl;
	return 0;
}
```

输出为

```
c is M and c's code is 77
now c is N and c's code is 78
```

char存储的是ANSI字符，那么对于字符数超过256个的字符集而言，就显得无能为力。

`wchar_t`表示的是宽字符，存储的是Unicode字符编码，用法为`wchar_t c = L'P'`,即要加上L前缀，占16位。

C++11新增了两种类型，为`char16_t`和`char32_t`，均为无符号类型，前者长16位，后者长12位，前者字面量要加上前缀小写u，后者要加上前缀大写U。

## 四、布尔型

这种很简单，关键字为`bool`。

## 五、空类型

关键字`void`定义的类型，不能用于普通变量的声明和操作，只能用于指针型变量，函数返回值和函数参数。

## 六、指针类型

指针是一种类型，该类型指向特定数据类型内存区的地址，即指针类型存放的是某一内存区的地址。

### 6.1 指针的声明

`typeName * pointerName`

指针的类型是什么，就代表该类型的指针指向的是什么类型的内存区。

### 6.2 指针的赋值

给指针赋值，方法有以下几种，分别是

+ 对于预定义类型而言，直接利用取址运算符&即可拿到特定类型的内存区地址，然后给指针变量即可；
+ 对于自定义类型而言，以数组为例，由于数组名代表的是数组的首元素的地址，所以直接将数组名赋给指针变量即可；
+ 上述两种情况是自动分配内存的且内存区具有代表该内存区的变量名，也可以使用new运算符为指针赋予特定类型内存区的匿名地址，要显式的通过`delete`来释放空间。

示例如下

```
int nights = 1001;
int * pt = &nights;    // 第一种方法, 或者如下一行
pt = &nights;
double wage[3] = {10000.0, 20000.0, 30000.0};
double * ptW = wage;    // 第二种方法，或者如下一行
double * ptN = &wage[0];    // 可见，指针要的只是数组首元素的地址
int * ps = new int;    // 第三种方法，创建int型的连续内存区域并将地址返回给指针
int * psome = new int[10];    // 第三种方法，创建int型的连续内存区域的数组，并将数组首元素的地址返回给指针
delete ps;    // 第三种方法，释放内存
delete [] psome;    // 第三种方法，释放内存
```

### 6.3 对指针解除引用

对指针解除引用意味着获取指针所指向内存区域的值。方法是通过*取值运算符，如下

```
cout << *pointerName
```

另外，指针中对数组的访问仍然可以使用数组下标的方式，即以下等式恒成立

```
arr[i] = *(ar + i);    // 数组访问方式中第i个元素的值为arr[i], 指针访问方式中第i个元素的地址为ar + i，那么值就是*(ar + i)
&arr[i] = ar + i;    // 数组访问方式中第i的元素的地址为&arr[i],指针访问方式中第i个元素的地址为ar + i。
```

### 6.4 区分指针和指针所指向的值

如果pt是指向int的指针，那么`*pt`不是指向int的指针，而是代表int型内存区域的值，也就是说效力等同于int型内存区域的变量名。只有pt才是指针，才代表的是某类型的地址，加上取值运算符`*`就成为了该地址的值。 示例如下

```
int * pt = new int;    // 开辟int大小的内存区域，并将地址给pt指针
*pt = 5；    // pt代表的值地址，那么*pt代表的就是值，这句代码就是将5存入pt地址处
```

### 6.5 指针算术

C++允许指针和整数相加，加1代表指向连续内存区域的下一个地址空间，减法也是同样的道理，所以可以通过`*(ar + i)`的形式访问数组，等同于`arr[i]`。

### 6.6 数组的动态联编和静态联编

使用数组声明来创建数组时，将采用静态联编，即数组的长度在编译时设置，如

```
int tacos[10];
```

使用`new`运算符创建数组时，将采用动态联编，即将在运行时为数组分配空间，其长度也将在运行时设置。这种情况下，应该通过`delete []`运算符显式释放内存。

### 6.7 指针和字符串

一般来说（比如以上），如果给cout提供的是指针，那么将打印地址，但是如果指针的类型为`char *`,那么cout打印指针指向的字符串。如果非要显式字符串的地址，那么需要将其强制转型为(int *)。如下所示

```
#include <iostream>

int main() {
	const char* pt = "wren";
	std::cout << pt << std::endl;
	std::cout << (int *)pt << std::endl;
	return 0;
}
```

输出为

```
wren
002F9FCC
```

由于字符串字面值是常量，所以被声明为`const`。

对于字符串的复制，不能简单粗暴的直接将字符串赋给指针，因为这样指针指向的依旧是以前的字符串，对指针的操作仍然会反映到字符串上，可行的办法就是通过`strcpy()`函数来实现，该函数接收两个参数，一个是目标地址，另一个是源地址。

## 七、自动存储、静态存储和动态存储

根据用于内存分配的方法，C++有三种管理数据内存的方式：自动存储、静态存储、动态存储。

### 7.1 自动存储

在函数内部定义的常规变量使用自动存储空间，被称为自动变量，这就意味着它们在所属的函数被调用时自动产生，在该函数结束时消亡。也就是说，自动变量是一个局部变量，其作用于仅为包含它的代码块。

自动变量通常存储在栈中，也就意味着执行代码块时，其中的变量将依次加载到栈中，而在离开代码块时，将按照相反的顺序释放这些变量。在程序执行过程中，栈将不断的增大或者缩小。

### 7.2 静态存储

静态存储是整个程序执行期间都存在的存储方式。使变量称为静态的方式有两种，分别是：

+ 一种是在函数外面定义它；
+ 另一种是在声明变量时使用关键字`static`修饰。

自动存储和静态存储的关键在于：这些方法严格地限制了变量的寿命。变量可能存在于程序的整个生命周期（静态变量），也可能只是在特定的函数被执行时存在（自动变量）。

### 7.3 动态存储

`new`和`delete`运算符提供了一种比自动变量和静态变量更为灵活的方法，它们管理一个内存池，这在C++中被称为自由存储空间(free store)或堆(heap)。`new`和`delete`运算符让你能够在一个函数中分配内存，在另一个函数中释放内存，因此，数据的生命周期不完全手程序或者函数的生存时间限制。示例如下

```
#include <iostream>

char* getName();

int main() {
	char* p = getName();
	std::cout << std::endl<< "Your name is " << p << " and memory address is " << (int *)p;
	std::cin.get();
	delete[] p;
	return 0;
}

char* getName() {
	char temp[80];
	std::cout << "Enter your name: ";
	std::cin >> temp;
	char* pn = new char[strlen(temp) + 1];
	strcpy(pn, temp);
	return pn;
}
```

运行结果为

```
Enter your name: Feily Zhang

Your name is Feily and memory address is 00B2D3D8
```