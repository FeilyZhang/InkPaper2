title: "C++中的预定义数据类型"
date: 2019-08-07 23:13:42 +0800
update: 2019-08-07 23:13:42 +0800
author: me
cover: "-/images/c++.jpg"
tags:
    - C++
preview: C++中的预定义数据类型包括整型、浮点型、字符型、布尔型、空类型、指针类型

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

