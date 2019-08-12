title: "深入C++函数"
date: 2019-08-12 16:27:42 +0800
update: 2019-08-12 16:27:42 +0800
author: me
cover: "-/images/c++.jpg"
tags:
    - C++
preview: 指针在函数中的应用。

---

## 一、C++内联函数

对常规函数的调用对应着跳转到标记函数起点的内存单元，然后执行函数代码，执行完毕后跳回到地址被保存的指令处。来回跳跃并记录跳跃位置是需要一定开销的。

C++提供的内联函数与常规函数不同，内联函数在编译阶段就将函数代码与调用者内联起来了，也就是使用函数代码替换了函数的调用，从而实现较快的执行速度，但是代价是需要额外的存储空间。使用方式为

+ 在函数声明前加上关键字`inline`;
+ 在函数定以前加上关键字`inline`。

```
#include <iostream>

inline void sayHello();

int main() {
	sayHello();
	return 0;
}

inline void sayHello() {
	std::cout << "hello, world!";
}
```

通常的做法是省略函数原型，将整个定义放在本应提供原型的地方，如下

```
#include <iostream>

inline void sayHello() { std::cout << "hello, world!"; }

int main() {
	sayHello();
	return 0;
}
```

## 二、引用变量

引用变量常用于函数的参数传递中，由于对于简单类型，C++采用的是值传递的方式，也就是函数体中的操作并不会影响被传入的参数的实际值，如果想实现简单类型的引用传递要么使用指针，要么使用引用变量。二者是有区别的，如下

+ 指针中`&`符号代表的是取址，而引用变量中`&`符号代表的是声明指向某具体类型的引用；
+ 指针中可以先声明然后赋值，但是引用变量只能创建是就要进行初始化；
+ 指针可以重复赋值，即改变指针单元的内存地址，但是引用变量一旦绑定就不可变了，但是并不意味着无法赋值，只是内存地址不变，赋值仍然会改变地址空间的值。

### 2.1 引用变量作为函数参数

用法如下

```
#include <iostream>

void show(int& i);

int main() {
	int i = 0;
	show(i);
	std::cout << "now i = " << i;
	return 0;
}

void show(int& i) {
	std::cout << "enter function i = " << i << "\n";
	std::cout << "exiting function i = " << ++i << "\n";
}
```

输出为

```
enter function i = 0
exiting function i = 1
now i = 1
```

可见，函数中对于引用变量的使用只是函数形参定义方式不同，其余均没什么变化。

### 2.2 引用变量的可变性与不可变性

以下程序作为对引用变量引用的验证

```
#include <iostream>

int main() {
	int i = 0;
	int& j = i;
	std::cout << "i = " << i << " j = " << j << "\n";
	std::cout << "i's address is " << &i << ", j's address is " << &j << "\n";
	int temp = 100;
	j = temp;
	std::cout << "i = " << i << " j = " << j << "\n";
	std::cout << "i's address is " << &i << ", j's address is " << &j;
	return 0;
}
```

输出为

```
i = 0 j = 0
i's address is 0032FA74, j's address is 0032FA74
i = 100 j = 100
i's address is 0032FA74, j's address is 0032FA74
```

在引用变量中，如果想让引用对变量的修改操作不生效，即只读，那么用`const`修饰即可。


### 2.3 函数返回引用变量

```
#include <iostream>

int& add(int& i, int& j);

int main() {
	int i = 1, j = 2;
	int& temp = add(i, j);
	std::cout << "temp's address is " << &temp << ", temp = " << temp << "\n";
	return 0;
}

int& add(int& i, int& j) {
	int& result = i;
	result = i + j;
	std::cout << "result's address is " << &result << ", result = " << result << "\n";
	return result;
}
```

输出为

```
result's address is 0037FA08, result = 3
temp's address is 0037FA08, temp = 3
```

这种情况下，其实质就是函数中将运算结果保存在一个内存空间中，然后将该内存地址返回给调用者，从而调用者持有该内存地址的引用，自然而然的与原有的变量地址相同，值自然也相同。