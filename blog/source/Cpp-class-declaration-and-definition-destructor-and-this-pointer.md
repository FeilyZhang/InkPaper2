title: "C++类声明与定义，析构函数与this指针"
date: 2019-08-16 19:23:42 +0800
update: 2019-08-16 19:23:42 +0800
author: me
cover: "-/images/c++.jpg"
tags:
    - C++
preview: C++类的声明与定义、构造函数、析构函数以及this指针。

---

## 一、C++类的声明与定义

与函数的定义与声明相同，先在`.h`文件中声明类，字段及方法，然后在`.cpp`文件中提供类的定义，然后再使用。完整的类声明如下所示

先是类的声明

```
// calculate.h -- Calculate class interface
#ifndef CALCULATE
#define CALCULATE

class Calculate {
    private:
		int left;
		int right;
    public:
		void setLeft(int left);
		void setRight(int right);
		int getLeft();
		int getRight();
		int add();
		int sub();
		int mul();
		int div();
};
#endif
```

再是类的定义

```
// calculate.cpp implementing the Calculate class
#include <iostream>
#include "calculate.h"

void Calculate::setLeft(int l) {
	left = l;
}

void Calculate::setRight(int r) {
	right = r;
}

int Calculate::getLeft() {
	return left;
}

int Calculate::getRight() {
	return right;
}

int Calculate::add() {
	return left + right;
}

int Calculate::sub() {
	return left - right;
}

int Calculate::mul() {
	return left * right;
}

int Calculate::div() {
	return left / right;
}
```

最后是类的使用，如下

```
#include <iostream>
#include "calculate.h"

int main() {
	using namespace std;
	Calculate cal;
	cal.setLeft(10);
	cal.setRight(100);
	cout << "left num  : " << cal.getLeft() << "\n";
	cout << "right num : " << cal.getRight() << "\n";
	cout << "two num's add : " << cal.add() << "\n";
	cout << "two num's sub : " << cal.sub() << "\n";
	cout << "two num's mul : " << cal.mul() << "\n";
	cout << "two num's div : " << cal.div() << "\n";
	return 0;
}
```

输出为

```
left num  : 10
right num : 100
two num's add : 110
two num's sub : -90
two num's mul : 1000
two num's div : 0
```

可以看出访问控制由`private`与`public`提供统一管理，既可以是字段也可以是方法，默认的访问权限是`private`。

在类的定义中，通过`Class::field/method()`的形式来定义字段与方法。

在类的使用上，与Java不同的是，Java需要通过`new`来创建对象，而C++中声明即创建对象，直接就可以使用。

## 二、构造函数与析构函数

C++同样可以提供一次性赋值的构造函数，默认的无参构造函数可以省略，但是如果定义了非默认构造函数那么一定要声明默认构造函数，否则会报错；

由于C++可能采用动态内存管理，所以需要一种在类使用完毕统一释放内存或其他资源的方式，这里是通过析构函数来实现的，基于上述代码，添加了构造函数与析构函数的声明与定义如下

```
// calculate.h -- Calculate class interface
#ifndef CALCULATE
#define CALCULATE

class Calculate {
    private:
		int left;
		int right;
    public:
		Calculate();
		Calculate(int l, int r);
		~Calculate();
		void setLeft(int left);
		void setRight(int right);
		int getLeft();
		int getRight();
		int add();
		int sub();
		int mul();
		int div();
};
#endif
```

```
// calculate.cpp implementing the Calculate class
#include <iostream>
#include "calculate.h"

Calculate::Calculate() {};

Calculate::Calculate(int l, int r) {
	left = l;
	right = r;
}

Calculate::~Calculate() {
	std::cout << "bye...";
}

void Calculate::setLeft(int l) {
	left = l;
}

void Calculate::setRight(int r) {
	right = r;
}

int Calculate::getLeft() {
	return left;
}

int Calculate::getRight() {
	return right;
}

int Calculate::add() {
	return left + right;
}

int Calculate::sub() {
	return left - right;
}

int Calculate::mul() {
	return left * right;
}

int Calculate::div() {
	return left / right;
}
```

```
#include <iostream>
#include "calculate.h"

int main() {
	using namespace std;
	Calculate cal;
	cal.setLeft(10);
	cal.setRight(100);
	cout << "left num  : " << cal.getLeft() << "\n";
	cout << "right num : " << cal.getRight() << "\n";
	cout << "two num's add : " << cal.add() << "\n";
	cout << "two num's sub : " << cal.sub() << "\n";
	cout << "two num's mul : " << cal.mul() << "\n";
	cout << "two num's div : " << cal.div() << "\n";
	
	Calculate cal1(100, 200);
	cout << "left num  : " << cal1.getLeft() << "\n";
	cout << "right num : " << cal1.getRight() << "\n";
	cout << "two num's add : " << cal1.add() << "\n";
	cout << "two num's sub : " << cal1.sub() << "\n";
	cout << "two num's mul : " << cal1.mul() << "\n";
	cout << "two num's div : " << cal1.div() << "\n";
	return 0;
}
```

输出为

```
left num  : 10
right num : 100
two num's add : 110
two num's sub : -90
two num's mul : 1000
two num's div : 0
left num  : 100
right num : 200
two num's add : 300
two num's sub : -100
two num's mul : 20000
two num's div : 0
bye...bye...
```

## 三、this指针

this指针代表类当前对象的引用，也就是说返回this指针就代表着返回当前类对象，如下

给头文件添加如下函数并给`getLeft()`方法添加`const`关键字

```
int getLeft() const;
const Calculate & compare(const Calculate & cal) const;
```

cpp文件如下

```
int Calculate::getLeft() const {
	return left;
}

const Calculate& Calculate::compare(const Calculate & cal) const {
	return left > cal.getLeft() ? *this : cal;
}
```

`compare()`方法通过比较两个对象的`left`字段，然后返回最大值的对象引用，主文件通过显示最大值来反映返回的引用是哪个对象的，如下

```
cout << "compare left value : " << cal1.compare(cal).getLeft() << "\n";
```

需要注意的是，在this指针中，如果一个函数调用了另一个函数，那么不仅要在调用者函数后面加上const关键字，还要在被调用函数要在参数列表后加上`const`关键字。